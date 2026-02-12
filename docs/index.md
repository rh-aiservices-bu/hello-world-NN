# OpenShift AI 3.x Workshop Showroom

This site walks you through a CPU-friendly lab in a **Red Hat OpenShift AI 3.x sandbox**:

1. Launch a **Workbench** using the **default PyTorch notebook image** in OpenShift AI and clone this repo via the JupyterLab Git UI.
2. Run a **PyTorch MNIST training notebook** in the Workbench.
3. Export the trained model to **ONNX format**.
4. Deploy and serve the model on **OpenShift AI model serving** using the **OpenVINO Model Server (OVMS)** runtime.
5. Test inference via the **KServe v2 API** and (optionally) from your laptop with `curl`.

> This is a *showroom*: it does **not** set up the cluster for you. It assumes you already have access to an OpenShift AI sandbox.

---

## What you’ll build
A simple digit classifier trained on MNIST. It’s small enough to train on CPU in a workshop setting.

![Sample MNIST digits](assets/firsteightimages.jpg)

## Lab navigation
Use the left nav to step through the lab.
