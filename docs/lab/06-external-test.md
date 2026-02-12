# Optional — Call the endpoint from your laptop

## Goal
Verify the model can be invoked from outside the cluster using `curl`.

## Prerequisites
- A Route was created in Step 3 (if not, create one now — see below)
- `curl` installed on your laptop

## 1. Verify the Route exists

```bash
oc get route mnist-onnx-inference -n <PROJECT_NAME>
```

If no Route exists, create one:

```bash
oc create route edge mnist-onnx-inference \
  --service=mnist-onnx-metrics \
  --port=8888 \
  -n <PROJECT_NAME>
```

!!! warning
    The Route must target `mnist-onnx-metrics` (not `mnist-onnx-predictor`). The predictor service is headless and Routes won't work against it.

## 2. Get the inference URL

```bash
export MODEL_URL="https://$(oc get route mnist-onnx-inference -n <PROJECT_NAME> -o jsonpath='{.spec.host}')"
echo $MODEL_URL
```

This gives you a URL like:

```
https://mnist-onnx-inference-<PROJECT_NAME>.apps.<CLUSTER_DOMAIN>
```

## 3. Check the model is reachable

```bash
curl -sk "$MODEL_URL/v2/models/mnist-onnx"
```

Expected response:

```json
{
  "name": "mnist-onnx",
  "versions": ["1"],
  "platform": "OpenVINO",
  "inputs": [{"name": "input", "datatype": "FP32", "shape": [-1,1,28,28]}],
  "outputs": [{"name": "output", "datatype": "FP32", "shape": [-1,10]}]
}
```

## 4. Send an inference request

```bash
curl -sk -X POST "$MODEL_URL/v2/models/mnist-onnx/infer" \
  -H "Content-Type: application/json" \
  -d '{
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
}'
```

## 5. Interpret the result

The `data` array in the response has 10 values. The index of the highest value is the predicted digit. For the vertical-line input above, the model should predict **1**.

## Troubleshooting
- **Connection refused / timeout**: the Route may not be externally accessible. Verify with `oc get route -n <PROJECT_NAME>` and confirm the hostname resolves with `nslookup` or `dig`.
- **TLS certificate errors**: use `-k` to skip certificate verification. The Route uses `edge` TLS termination with the cluster's default certificate, which is often self-signed.
- **401 / 403**: some sandboxes require authentication headers. Check with your workshop staff.
- **Empty response or HTML error page**: confirm the Route targets `mnist-onnx-metrics` on port `8888`, not the predictor service.
