# Reference / Appendix

## Repo assets
- `mnist_sequential.ipynb` — training notebook; attendees clone this repo via the JupyterLab Git UI to use it.
- `img/` — images used by the notebook and docs.
- `Containerfile` — reference only; shows how a custom Workbench image could be assembled (not used in this lab since attendees use the default PyTorch image).
- `openshift/buildconfig-pytorch.yaml` and `openshift/imagestream-pytorch.yaml` — example OpenShift build assets (reference only, not used in the lab).

## Model serving details (validated)
- **Model format**: ONNX (exported from PyTorch via `torch.onnx.export`, opset 13)
- **Model file**: `mnist_model.onnx` (~430 KB)
- **Serving runtime**: OpenVINO Model Server (OVMS) — template `kserve-ovms`
- **Input**: `float32` tensor, shape `[batch, 1, 28, 28]`, name `input`
- **Output**: `float32` tensor, shape `[batch, 10]`, name `output` (raw logits, argmax = predicted digit)
- **API**: KServe v2 REST — `POST /v2/models/mnist-onnx/infer`

## Notes on the Gen AI playground
The OpenShift AI documentation describes the Gen AI playground as a feature to evaluate and interact with foundation and custom generative models, with optional RAG and MCP server integrations. Use it for LLM- and chat-style endpoints, not classic MNIST-style inference.
