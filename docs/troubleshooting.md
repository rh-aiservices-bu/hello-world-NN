# Troubleshooting

## Workbench does not start
- Confirm you selected the **PyTorch | CUDA | 3.12** notebook image from the standard images list (not a custom or imported image).
- Confirm your project has sufficient resource quota for the Workbench pod.

## Notebook missing after launch
After the Workbench starts, you need to clone the workshop repository using the JupyterLab GUI:

1. Click **Git → Clone a Repository** from the top menu bar.
2. Paste the repository URL and click **Clone**.

- If the clone fails due to network restrictions, ask your workshop staff for an alternative (e.g., direct file upload or a pre-populated PVC).

## Training issues

### Kernel dies or out of memory
- Reduce batch size in the `DataLoader` cells: change `batch_size=32` to `batch_size=16`.
- The model has only 109,386 parameters and should train comfortably on CPU with 512 MB of memory.

### ONNX export warning
You may see: `DeprecationWarning: You are using the legacy TorchScript-based ONNX export`. This is harmless — the export still works correctly.

## Model serving fails to start

### Helper pod fails with SCC / security context error
If you see an error like `pods "model-upload" is forbidden: unable to validate against any security context constraint`, the sandbox has restrictive security policies. Common causes:

- **`fsGroup: 0`** — sandbox SCCs restrict `fsGroup` to a project-specific range. Remove `fsGroup` entirely; OpenShift automatically assigns a valid group that makes the PVC writable.
- **`runAsUser: 0`** — running as root is not allowed. The helper pod does not need root.
- **Init containers with privileged settings** — remove any init containers with `securityContext.runAsUser: 0`.

The pod spec in the lab instructions works with the default `restricted-v2` SCC without any special security settings.

### Pod stuck in ContainerCreating
- **Multi-Attach error**: the PVC is still attached to another pod (e.g., your Workbench or the upload pod). PVCs with `ReadWriteOnce` can only be mounted by one pod at a time. Delete the upload pod first: `oc delete pod model-upload -n $PROJECT_NAME`.
- **Image pull errors**: the OVMS image is pulled from `quay.io/modh/openvino_model_server:stable`. Confirm the cluster has network access to Quay.io.

### CrashLoopBackOff — "File not found"
Check the OVMS pod logs:

```bash
oc logs -l app=mnist-onnx -n $PROJECT_NAME
```

Common causes:

- **Wrong directory layout**: OVMS expects a version directory. The correct layout is:
  ```
  <storage-root>/1/model.onnx
  ```
- **Wrong file name**: the file must be named exactly `model.onnx`.
- **`lost+found` warning**: you may see `Expected version directory name to be in number format. Got: lost+found` repeated every second in the logs. This is harmless and can be ignored — the model will still load and serve correctly.

### Deployment has 0 ready replicas
```bash
oc get deployment mnist-onnx -n $PROJECT_NAME
```

Check the pod status:
```bash
oc get pods -l app=mnist-onnx -n $PROJECT_NAME
```

If no pods exist, verify the Deployment was created successfully. If pods are stuck, check their logs and events for errors.

## Inference endpoint issues

### Connection refused on port-forward
Verify you're using the correct service name:

```bash
oc port-forward svc/mnist-onnx -n $PROJECT_NAME 8888:8888
```

### 404 on inference endpoint
- Verify the model name in the URL matches: `/v2/models/mnist-onnx/infer`
- Check model metadata first: `curl -k https://<ROUTE_URL>/v2/models/mnist-onnx`
- If the metadata endpoint returns valid JSON, the model is loaded and the issue is with your inference request format.

### Endpoint reachable inside cluster but not outside
- Confirm a Route exists: `oc get route -n $PROJECT_NAME`
- If no Route exists, create one:
  ```bash
  oc create route edge mnist-onnx --service=mnist-onnx --port=8888 -n $PROJECT_NAME
  ```
- Check if the sandbox restricts external ingress.

### Inference returns incorrect predictions
- Verify input data is preprocessed correctly (normalized, correct shape).
- Test with the provided `payload.json` example first — it should predict digit "1".
- Check that the model file on the PVC matches the one you exported from the notebook.
