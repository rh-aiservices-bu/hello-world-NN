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
If you see an error like `pods "model-upload" is forbidden: unable to validate against any security context constraint`, the sandbox may have restrictive security policies. The helper pod does not require any special privileges — make sure you are not using an init container with `runAsUser: 0`. The pod spec in the lab instructions should work with the default `restricted` SCC.

### Pod stuck in ContainerCreating
- **Multi-Attach error**: the PVC is still attached to another pod (e.g., your Workbench or the upload pod). PVCs with `ReadWriteOnce` can only be mounted by one pod at a time. Delete the upload pod first: `oc delete pod model-upload -n <PROJECT_NAME>`.
- **Image pull errors**: the OVMS image is pulled from `registry.redhat.io`. Confirm the cluster has valid pull credentials.

### CrashLoopBackOff — "File not found"
Check the OVMS pod logs:

```bash
oc logs -l serving.kserve.io/inferenceservice=mnist-onnx -n <PROJECT_NAME>
```

Common causes:

- **Wrong directory layout**: OVMS expects a version directory. The correct layout is:
  ```
  <storage-root>/1/model.onnx
  ```
- **Wrong file name**: the file must be named exactly `model.onnx`.
- **`lost+found` warning**: you may see `Expected version directory name to be in number format. Got: lost+found` — this is harmless and can be ignored.

### InferenceService stays READY=False
```bash
oc get inferenceservice mnist-onnx -n <PROJECT_NAME> -o yaml
```

Check the `status.conditions` for error messages. Common issues:
- Serving runtime `kserve-ovms` doesn't exist in the namespace — create it from the template first.
- Storage URI is incorrect — verify the PVC name or S3 bucket path.

## Inference endpoint issues

### Connection refused on port-forward
The predictor service is headless (ClusterIP: None). Use the metrics service instead:

```bash
oc port-forward svc/mnist-onnx-metrics -n <PROJECT_NAME> 8888:8888
```

### 404 on inference endpoint
- Verify the model name in the URL matches: `/v2/models/mnist-onnx/infer`
- Check model metadata first: `curl http://localhost:8888/v2/models/mnist-onnx`

### Endpoint reachable inside cluster but not outside
- Confirm a Route exists: `oc get route -n <PROJECT_NAME>`
- If no Route exists, create one:
  ```bash
  oc expose svc/mnist-onnx-metrics -n <PROJECT_NAME> --port=8888
  ```
  Or for TLS:
  ```bash
  oc create route edge mnist-onnx-inference \
    --service=mnist-onnx-metrics \
    --port=8888 \
    -n <PROJECT_NAME>
  ```
- Check if the sandbox restricts external ingress.
