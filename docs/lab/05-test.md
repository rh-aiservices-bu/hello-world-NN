# Step 4 — Test the inference endpoint

## Goal
Send requests to the deployed MNIST model and verify it returns correct digit predictions.

## Important note about the Gen AI playground
OpenShift AI's **Gen AI playground** is built for interacting with *generative/chat-style models* (foundation or custom) that are exposed as **AI asset endpoints**.

For a **MNIST classifier served via OVMS**, the playground will not work. Use the **KServe v2 inference API** instead:

- `curl` to the v2 REST endpoint
- A small Python client in the notebook

![Locating the inference endpoint URL in the OpenShift AI UI](../assets/placeholder-inference-endpoint-url.png)
<!-- TODO (workshop author): Replace with a screenshot showing where to find the model's inference endpoint URL in the dashboard -->

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
    export MODEL_URL="https://$(oc get route mnist-onnx-inference -n <PROJECT_NAME> -o jsonpath='{.spec.host}')"
    ```

=== "Port-forward (no Route)"

    If no Route is available, use port-forwarding. **Use the metrics service, not the predictor service:**

    ```bash
    oc port-forward svc/mnist-onnx-metrics -n <PROJECT_NAME> 8888:8888
    ```

    Then in a second terminal:

    ```bash
    export MODEL_URL="http://localhost:8888"
    ```

    !!! warning
        Do **not** port-forward to `svc/mnist-onnx-predictor` — it is a headless service and port-forwarding to it will fail with `connection refused`.

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

Create a file called `payload.json`. Here's a minimal example — a vertical line that the model should recognize as the digit **1**:

```json
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
```

Send the request:

```bash
curl -sk -X POST "$MODEL_URL/v2/models/mnist-onnx/infer" \
  -H "Content-Type: application/json" \
  -d @payload.json
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

# Use the metrics service URL (port 8888 inside the cluster)
url = "http://mnist-onnx-metrics.<PROJECT_NAME>.svc.cluster.local:8888/v2/models/mnist-onnx/infer"

# Pick a test image from the dataset
image, true_label = test_dataset[0]
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

!!! note "Service URL in the Python client"
    Use the **metrics service** URL (`mnist-onnx-metrics`) on port **8888** from within the cluster. The predictor service is headless and may not resolve reliably from all pods.

## Success criteria
- HTTP 200 responses with valid JSON output.
- The predicted digit matches the input image.
- Predictions are stable across repeated calls for the same input.

![Successful inference response](../assets/placeholder-successful-inference.png)
<!-- TODO (workshop author): Replace with a screenshot or terminal output showing a successful prediction response -->
