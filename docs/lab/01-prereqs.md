# Prerequisites

## Accounts and access
- Access to an **OpenShift AI 3.x** environment (sandbox).
- Permission to **create a project** and **create a Workbench** in that project.
- Permission to **deploy a model** on the model-serving platform in that project.
- **Outbound internet access** from the Workbench (needed to clone the repo and download MNIST data).

![OpenShift AI dashboard landing page](oai-dashboard.png)

## Tools
- **`oc` CLI** — logged in to the cluster. You will use `oc` to upload the model to storage, create the serving runtime, and (optionally) create a Route.
- **OpenShift AI dashboard** — used to deploy the model to the serving platform.
- **`curl`** — only needed for the optional external test step.

## Values you will need (fill these in)
Create a sticky note with these items:

| Value | Example | Where it's used |
|-------|---------|-----------------|
| Cluster console URL | `https://console-openshift-console.apps...` | Accessing the UI |
| OpenShift AI dashboard URL | `https://rhods-dashboard...` | Workbench creation |
| Your project name | `hello-world-nn` | All `oc` commands |
| Workshop Git repo URL | `https://github.com/...` | Cloning into Workbench |

!!! note "Project naming"
    Choose a short, lowercase name with no special characters. You will type it repeatedly in `oc` commands.
