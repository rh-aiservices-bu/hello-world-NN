# Claude task list (workshop author)

This repo currently contains a **skeleton** MkDocs site. Use this checklist to have Claude Code fill in the missing pieces.

## P0 — Make the docs build and deploy
1. Verify `mkdocs build` works locally.
2. Verify the GitHub Pages workflow in `.github/workflows/pages.yml` works with repository settings:
   - Pages → Source: **GitHub Actions**
3. Replace placeholders in `mkdocs.yml`:
   - `site_url`
   - `repo_url`

## P1 — Populate the lab with concrete steps
Fill in every `TODO` block in `docs/lab/*.md` with sandbox-appropriate steps:
- How to create a project in OpenShift AI
- How to launch a Workbench using the default PyTorch notebook image and clone the repo via the JupyterLab Git UI
- How to run the notebook and export artifacts
- How to create a model serving deployment using the PyTorch runtime
- How to test inference from:
  - OpenShift AI UI (if available)
  - Notebook Python client
  - (Optional) laptop `curl`

## P1 — Use and verify real artifacts
1. Open `mnist_sequential.ipynb` and identify exactly:
   - what file(s) it writes out
   - where it writes them
   - what input format the served runtime expects
2. Update the docs to match reality (no guessed filenames).

## P2 — Remove sandbox foot-guns
- Avoid any step that requires cluster-admin.
- If a step depends on a feature that might be disabled (e.g., external Routes), label it clearly as **Optional**.

## P2 — Quality and polish
- Add screenshots (optional) and keep pages skimmable.
- Add a 1-page “What good looks like” verification checklist.

