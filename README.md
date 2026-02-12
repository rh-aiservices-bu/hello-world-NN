# OpenShift AI 3.x Workbench → PyTorch Serving Workshop (Showroom)

This repository is the **documentation/showroom** for a CPU-friendly OpenShift AI 3.x workshop.

## What's in here
- `docs/` + `mkdocs.yml`: GitHub Pages site (MkDocs Material)
- `mnist_sequential.ipynb` + `img/`: training notebook and assets (attendees clone this repo via the JupyterLab Git UI)
- `Containerfile`: reference only (not used in the lab — attendees use the default PyTorch notebook image)
- `openshift/`: reference OpenShift build manifests (not used in the lab)

## Local preview
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

## Publish
Push to `main`. GitHub Actions publishes to GitHub Pages.

> Repo settings: Settings → Pages → Source: **GitHub Actions**.
