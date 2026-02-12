# Authoring guide

## How to use this repo with Claude Code
Run Claude Code at the repo root and give it a single objective at a time, for example:
- “Fill in Step 1 and Step 2 pages using the assets in this repo and OpenShift AI 3.0 docs.”
- "Add a troubleshooting section for common Workbench startup issues in OpenShift AI."

## Documentation conventions
- Use short numbered steps.
- Use **copy/paste blocks** for all commands.
- Prefer UI click paths like:
  `OpenShift AI → Data science projects → <project> → Workbenches → Create`.
- Keep CPU-first instructions (small batch size, fewer epochs).

## What’s intentionally left as TODO
Any time you see **TODO (workshop author)**, Claude should replace it with:
- concrete UI steps
- concrete values (or clearly stated placeholders)
- screenshots (optional)
