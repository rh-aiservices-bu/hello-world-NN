# Claude Code Instructions (OpenShift AI 3.x Workshop Showroom)

You are working inside the repository **hello-world-NN**.

## What you are building
Create a **static documentation site** (GitHub Pages) that guides attendees through a hands-on flow in a **Red Hat OpenShift AI 3.x sandbox** (no pre-provisioned lab automation).

This is **not** a full lab environment repo. The deliverable is the *showroom/tutorial website* plus minimal supporting repo files.

## Workshop flow to document
1) **Launch a Workbench** using the **default PyTorch notebook image** provided by OpenShift AI 3.x, then use the JupyterLab Git UI to clone this repo for the notebook and assets.
2) **Run training/experiments** in the Workbench (Jupyter).
3) **Deploy the trained model** (exported to ONNX) from the Workbench to **OpenShift AI 3.x model serving** using the **OpenVINO Model Server (OVMS)** runtime.
4) **Experiment with the deployment** (UI + basic inference testing). Note: the *Gen AI playground* is primarily for generative/chat-style models, not classic ML inference.
5) *(Optional)* Verify the model endpoint works externally (attendee runs `curl` from their laptop).

## Hard constraints
- Assume **no significant GPU** capacity.
- Assume attendees have access to an **OpenShift AI sandbox** and basic permissions (create project, workbench, model deployment). Avoid instructions that require cluster-admin or image-building RBAC.
- Use placeholders for any values we cannot know (cluster URL, Quay org, model route).

## Repo facts you must use
The repo already contains:
- `mnist_sequential.ipynb` and `img/` — attendees use the JupyterLab Git UI to clone this repo into their Workbench.
- `Containerfile` — reference only; the lab does **not** build a custom image (attendees lack RBAC for that).
- `openshift/buildconfig-pytorch.yaml`, `openshift/imagestream-pytorch.yaml`, and `openshift/deploy.sh` — reference assets for advanced users only.

## Output format expectations
- Use **MkDocs + Material** to generate the site.
- Add a GitHub Actions workflow that publishes to GitHub Pages.
- Docs must be easy to skim: numbered steps, callouts, and copy/paste blocks.
- Keep commands short and avoid deep YAML unless it’s essential.

## Content quality bar
- Every step should include:
  - what the attendee is doing
  - why it matters
  - what "success" looks like
  - common mistakes / troubleshooting

## Decisions you should make (without asking)
- Prefer `mkdocs-material` with a minimal navigation tree.
- Use a simple, consistent page structure:
  - Overview
  - Prereqs
  - Lab Steps (Workbench → Train → Export ONNX → Serve with OVMS → Test via v2 API)
  - Troubleshooting
  - Reference / Appendix

## Specific callout to include
The OpenShift AI **Gen AI playground** is designed for interacting with *foundation and custom generative models* via AI asset endpoints. In this workshop we deploy a **PyTorch MNIST classifier**, so the primary testing path should be:
- OpenShift AI model serving UI “Test” (if available for that endpoint type)
- `curl` to the inference route
- A tiny Python client in the notebook

(You can optionally add a sidebar note explaining how to use the playground when the served model is an LLM.)

## Minimal acceptance checklist
- `mkdocs.yml` exists and builds locally (`mkdocs build`).
- `docs/` contains the full lab outline and placeholders.
- `.github/workflows/` includes a Pages deploy workflow.
- The site renders images from `docs/assets/`.

