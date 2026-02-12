# Glossary

- **Workbench**: A Jupyter-based development environment in OpenShift AI.
- **Model Serving**: The platform capability to deploy models as inference services.
- **Serving Runtime**: The container/runtime implementation used to load and serve a model. This lab uses **OpenVINO Model Server (OVMS)**.
- **OVMS (OpenVINO Model Server)**: An inference server that supports ONNX, OpenVINO IR, TensorFlow, and PyTorch models. Used as the serving runtime in this lab.
- **ONNX (Open Neural Network Exchange)**: A portable, framework-agnostic model format. The notebook exports the trained PyTorch model to ONNX for serving.
- **KServe v2 protocol**: The standardized REST API used by OVMS and other runtimes for inference requests (`/v2/models/<name>/infer`).
- **InferenceService**: A KServe custom resource that defines a model deployment (model format, runtime, storage location, resources).
- **AI asset endpoint**: A model endpoint registered for use in Gen AI Studio features (primarily generative models).
