# Step 3 — Deploy to model serving

## Goal
Upload your trained MNIST model to cluster storage and deploy it using the **OpenVINO Model Server (OVMS)** runtime.

## Mental model
```text
model.onnx (exported by the notebook)
  └─> Download to your laptop
        └─> Upload to PVC under 1/model.onnx via oc CLI
              └─> Deploy OVMS Deployment with PVC mounted
                    └─> OVMS loads model and exposes v2 API on port 8888
                          └─> Route exposes the endpoint externally
```

- Your **ONNX model artifact** lives on a PVC in the versioned directory structure OVMS expects: `1/model.onnx`
- The **OVMS serving runtime** loads the model and exposes a **v2 inference API** on port **8888**
- A **Route** makes the API accessible from outside the cluster

!!! info "Why not use the OpenShift AI dashboard?"
    The OpenShift AI dashboard UI expects S3-compatible storage (like AWS S3 or MinIO) for model deployment. In sandbox environments without S3 access, the simpler approach is to deploy OVMS directly as a standard Deployment using a PVC. This gives you the same inference capabilities without the extra complexity.

## Procedure

### 1. Download the model from the Workbench

In JupyterLab, locate the `model.onnx` file in the file browser. Right-click the file and select **Download** to save it to your laptop.

![Downloading the model from JupyterLab](../assets/download-model.png)

You will upload this file to cluster storage in the next step.

### 2. Upload the model to a PVC

OVMS expects the model file inside a **numbered version directory**. The final layout on the PVC must be:

```
mnist-model-pvc (PersistentVolumeClaim)
  └─ 1/
      └─ model.onnx
```

!!! danger "File naming is strict"
    OVMS will **crash with `File not found`** if:

    - The file is not named exactly `model.onnx`
    - There is no version directory (`1/`)
    - The file is placed directly at the PVC root without a version directory

    You may also see a harmless warning about `lost+found` if using a PVC — this can be ignored.

First, create the PVC:

```bash
cat <<EOF | oc apply -n <PROJECT_NAME> -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mnist-model-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF
```

Next, create a temporary helper pod that mounts the PVC so you can copy the model onto it:

```bash
cat <<EOF | oc apply -n <PROJECT_NAME> -f -
apiVersion: v1
kind: Pod
metadata:
  name: model-upload
spec:
  containers:
  - name: upload
    image: registry.access.redhat.com/ubi9/ubi:latest
    command: ["sh", "-c", "mkdir -p /mnt/models/1 && sleep 3600"]
    volumeMounts:
    - name: model-storage
      mountPath: /mnt/models
  volumes:
  - name: model-storage
    persistentVolumeClaim:
      claimName: mnist-model-pvc
  restartPolicy: Never
EOF
```

Wait for the pod, copy the model, verify, then clean up:

```bash
# Wait for the pod to be ready
oc wait pod/model-upload -n <PROJECT_NAME> --for=condition=Ready --timeout=120s

# Copy model.onnx into the version directory on the PVC
oc cp model.onnx <PROJECT_NAME>/model-upload:/mnt/models/1/model.onnx -c upload

# Verify the file is in place
oc exec model-upload -n <PROJECT_NAME> -c upload -- ls -la /mnt/models/1/

# Delete the upload pod to release the PVC
oc delete pod model-upload -n <PROJECT_NAME>
```

!!! warning "Delete the upload pod before deploying"
    The PVC uses `ReadWriteOnce`, which means only one pod can mount it at a time. **You must delete the upload pod before the serving pod can start.** If you see a `Multi-Attach error`, this is why.

### 3. Deploy the model serving runtime

Create a Deployment that runs OVMS with your PVC mounted:

```bash
cat <<EOF | oc apply -n <PROJECT_NAME> -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mnist-onnx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mnist-onnx
  template:
    metadata:
      labels:
        app: mnist-onnx
    spec:
      containers:
      - name: ovms
        image: quay.io/modh/openvino_model_server:stable
        args:
        - --model_name=mnist-onnx
        - --model_path=/mnt/models
        - --port=8001
        - --rest_port=8888
        ports:
        - containerPort: 8888
          name: http
        volumeMounts:
        - name: model-storage
          mountPath: /mnt/models
      volumes:
      - name: model-storage
        persistentVolumeClaim:
          claimName: mnist-model-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mnist-onnx
spec:
  selector:
    app: mnist-onnx
  ports:
  - port: 8888
    targetPort: 8888
    name: http
EOF
```

This creates:
- A **Deployment** with 1 replica running OVMS
- A **Service** exposing port 8888

### 4. Create an external Route

Create a Route to expose the inference endpoint outside the cluster:

```bash
oc create route edge mnist-onnx --service=mnist-onnx --port=8888 -n <PROJECT_NAME>
```

### 5. Verify the deployment

Check that the pod is running:

```bash
oc get pods -n <PROJECT_NAME> -l app=mnist-onnx
```

Expected output:

```
NAME                          READY   STATUS    RESTARTS   AGE
mnist-onnx-67d78c4887-xxxxx   1/1     Running   0          30s
```

Check the logs to confirm the model loaded:

```bash
oc logs -l app=mnist-onnx -n <PROJECT_NAME> | head -50
```

You should see output like:

```
Loading model: mnist-onnx, version: 1, from path: /mnt/models/1, with target device: CPU ...
Input name: input; mapping_name: input; shape: (-1,1,28,28); precision: FP32; layout: N...
Output name: output; mapping_name: output; shape: (-1,10); precision: FP32; layout: N...
Loaded model mnist-onnx; version: 1; batch size: -1; No of InferRequests: 1
STATUS CHANGE: Version 1 of model mnist-onnx status change. New status: ( "state": "AVAILABLE", "error_code": "OK" )
```

!!! note "`lost+found` warnings"
    You'll see recurring warnings about `lost+found` in the logs. This is harmless — PVC formatting creates this directory, and OVMS logs a warning every time it scans for model versions. The model still loads and serves correctly.

Test the metadata endpoint:

```bash
curl -k https://$(oc get route mnist-onnx -n <PROJECT_NAME> -o jsonpath='{.spec.host}')/v2/models/mnist-onnx
```

Expected output:

```json
{"name":"mnist-onnx","versions":["1"],"platform":"OpenVINO","inputs":[{"name":"input","datatype":"FP32","shape":[-1,1,28,28]}],"outputs":[{"name":"output","datatype":"FP32","shape":[-1,10]}]}
```

## What "success" looks like
- The pod shows `Running` with `READY 1/1`
- Logs show `STATUS CHANGE: ... "state": "AVAILABLE"`
- The metadata endpoint returns JSON with model input/output shapes
- No CrashLoopBackOff or image pull errors

## Common issues
- **Helper pod fails with SCC error**: verify you are not using `runAsUser: 0`, `fsGroup: 0`, or any privileged settings. The pod spec in these instructions works with the default `restricted-v2` SCC.
- **Pod stuck in `ContainerCreating`**: check if the PVC is still attached to the upload pod. Delete it first: `oc delete pod model-upload -n <PROJECT_NAME>`.
- **CrashLoopBackOff — "File not found"**: the directory layout is wrong. Verify `1/model.onnx` exists on the PVC by re-running the upload steps.
- **`lost+found` warning spam in logs**: harmless — OVMS logs this every second but the model still works.
- **Image pull errors**: the OVMS image is pulled from `quay.io/modh/openvino_model_server:stable` and should be publicly accessible. If you see pull errors, check your cluster's image registry connectivity.
- **Multi-Attach error**: the upload pod is still running. Delete it: `oc delete pod model-upload -n <PROJECT_NAME>`.
