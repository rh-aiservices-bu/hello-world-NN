# Step 4 — Test the inference endpoint

## Goal
Send requests to the deployed MNIST model and verify it returns correct digit predictions.

## Important note about the Gen AI playground
OpenShift AI's **Gen AI playground** is built for interacting with *generative/chat-style models* (foundation or custom) that are exposed as **AI asset endpoints**.

For a **MNIST classifier served via OVMS**, the playground will not work. Use the **KServe v2 inference API** instead:

- `curl` to the v2 REST endpoint
- A small Python client in the notebook

## API details

The OVMS serving runtime exposes the [KServe v2 inference protocol](https://kserve.github.io/website/latest/modelserving/data_plane/v2_protocol/) on **port 8888**:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v2/models/mnist-onnx` | GET | Model metadata (input/output shapes) |
| `/v2/models/mnist-onnx/infer` | POST | Run inference |

## 1. Get your inference URL

=== "External Route"

    If you created a Route in the previous step:

    ```bash
    export MODEL_URL="https://$(oc get route mnist-onnx -n $PROJECT_NAME -o jsonpath='{.spec.host}')"
    ```

=== "Port-forward (no Route)"

    If no Route is available, use port-forwarding:

    ```bash
    oc port-forward svc/mnist-onnx -n $PROJECT_NAME 8888:8888
    ```

    Then in a second terminal:

    ```bash
    export MODEL_URL="http://localhost:8888"
    ```

## 2. Check model metadata

Verify the model is loaded and inspect its input/output shapes:

```bash
curl -sk "$MODEL_URL/v2/models/mnist-onnx"
```

Response:

```json
{
  "name": "mnist-onnx",
  "versions": ["1"],
  "platform": "OpenVINO",
  "inputs": [{"name": "input", "datatype": "FP32", "shape": [-1,1,28,28]}],
  "outputs": [{"name": "output", "datatype": "FP32", "shape": [-1,10]}]
}
```

This tells you: the model expects a `float32` tensor of shape `[batch, 1, 28, 28]` and returns 10 logits (one per digit 0-9).

## 3. Send a test inference request

Send a test request with a simple vertical line image (which should be recognized as digit **1**):

```bash
curl -sk -X POST "$MODEL_URL/v2/models/mnist-onnx/infer" \
  -H "Content-Type: application/json" \
  -d @- <<'EOF'
{
  "inputs": [{
    "name": "input",
    "shape": [1, 1, 28, 28],
    "datatype": "FP32",
    "data": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
             0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
  }]
}
EOF
```

## 4. Interpret the response

The response contains 10 logits — one for each digit (0-9). The **highest value** is the model's prediction:

```json
{
  "model_name": "mnist-onnx",
  "model_version": "1",
  "outputs": [{
    "name": "output",
    "shape": [1, 10],
    "datatype": "FP32",
    "data": [-23.93, 9.55, -8.54, -9.35, -4.10, -6.75, -8.34, -3.78, -2.13, -9.00]
  }]
}
```

Reading the `data` array by index:

| Index | 0 | **1** | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|-------|-------|------|------|------|------|------|------|------|------|------|
| Logit | -23.9 | **9.55** | -8.5 | -9.4 | -4.1 | -6.7 | -8.3 | -3.8 | -2.1 | -9.0 |

Index **1** has the highest value (9.55), so the model predicts **digit "1"** — correct for our vertical line input.

## Test from the notebook (Python)

You can also test from within the Workbench by adding a cell to the notebook:

```python
import requests
import numpy as np
import random

# Get random digit selection to send to model
rands = random.randint(0, len(test_dataset)-1)

# Use the service URL (port 8888 inside the cluster)
# Replace <PROJECT_NAME> with your actual project name
url = "http://mnist-onnx.<PROJECT_NAME>.svc.cluster.local:8888/v2/models/mnist-onnx/infer"

# Pick a test image from the dataset
image, true_label = test_dataset[rands]
payload = {
    "inputs": [{
        "name": "input",
        "shape": [1, 1, 28, 28],
        "datatype": "FP32",
        "data": image.numpy().flatten().tolist()
    }]
}

response = requests.post(url, json=payload)
logits = response.json()["outputs"][0]["data"]
predicted = np.argmax(logits)
print(f"True: {true_label}, Predicted: {predicted}, Correct: {true_label == predicted}")
```

## Success criteria
- HTTP 200 responses with valid JSON output.
- The predicted digit matches the input image.
- Predictions are stable across repeated calls for the same input.
