# Optional — Cleanup

## Goal
Remove the workshop resources from your OpenShift project in case you want to experiment with something different in the future.

## What gets deleted
- Model serving deployment and service
- External route
- PVC containing the model
- (Optional) Workbench

## Cleanup commands

### 1. Delete the model serving resources

Delete the deployment and service:

```bash
oc delete deployment mnist-onnx -n $PROJECT_NAME
oc delete service mnist-onnx -n $PROJECT_NAME
```

### 2. Delete the route

```bash
oc delete route mnist-onnx -n $PROJECT_NAME
```

### 3. Delete the model storage PVC

```bash
oc delete pvc mnist-model-pvc -n $PROJECT_NAME
```

### 4. (Optional) Delete the Workbench

If you want to remove the Workbench as well:

**Via the OpenShift AI dashboard:**

1. Go to your Data Science Project
2. Click the **Workbenches** tab
3. Click the three-dot menu next to `mnist-workbench`
4. Select **Delete workbench**

**Via CLI:**

```bash
oc delete notebook mnist-workbench -n $PROJECT_NAME
```

!!! warning "This deletes your work"
    Deleting the Workbench will remove the Jupyter environment and any unsaved notebooks. If you want to keep the notebook, download `mnist_sequential.ipynb` from JupyterLab before deleting the Workbench.

## Verify cleanup

Check that resources are deleted:

```bash
oc get deployment,service,route,pvc -n $PROJECT_NAME
```

You should no longer see `mnist-onnx` resources listed.
