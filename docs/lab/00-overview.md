# Lab overview

## What you will build
You will train a simple **image classifier** (MNIST digits) inside an OpenShift AI Workbench, export it to ONNX format, then deploy it as an inference endpoint using the **OpenVINO Model Server (OVMS)** serving runtime in OpenShift AI 3.x.

## Why this lab exists
- Demonstrate the *Workbench → Model Serving* workflow without requiring GPUs.
- Show how to use the default PyTorch notebook image with a cloned repo — no custom image builds required.
- Give attendees a concrete, non-LLM example of OpenShift AI model serving.

## What you will need
- A project/namespace in OpenShift AI
- Outbound internet access from the Workbench (to download MNIST data and clone the repo)
- The `oc` CLI (for uploading the model to storage and creating the serving runtime)
- Access to the **OpenShift AI dashboard** (for deploying the model)

## Flow map
```text
Default PyTorch notebook image + cloned repo
  └─> Jupyter notebook trains model on CPU (~2 min)
        └─> model.onnx exported by notebook
              └─> Download model, upload to PVC via oc CLI (placed in 1/model.onnx)
                    └─> Deploy model through OpenShift AI dashboard
                          └─> OVMS serving runtime loads model (<1s)
                                └─> KServe v2 REST API on port 8888
                                      └─> Route exposes endpoint externally
                                            └─> curl / Python client test
```

!!! info "Key detail: directory structure"
    The notebook exports the model as `model.onnx`. OVMS expects this file inside a **numbered version directory** (`1/model.onnx`) on the PVC. The upload step handles placing it in the correct location.
