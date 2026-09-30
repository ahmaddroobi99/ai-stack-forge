# Contributing

## How to work here

1. Read `REQUIREMENTS.md` and `docs/SYSTEM_DESIGN.md`.
2. Pick one lane from the README table.
3. Open an issue named `lane/<name>: <change>`.
4. Keep upstream projects upstream. Put glue in this repo.
5. Do not enable Langflow, Flowise, and Dify all as sources of truth.

## Local rules

- Pin container tags.
- New ports go in `docs/STACK.md` and `.env.example`.
- Model calls go through the gateway once it exists.
- Do not commit weights, `.env`, or raw team documents.

## PR checklist

- [ ] Lane named in the PR title
- [ ] Contract change documented if you touched `/v1/*`
- [ ] Health check still explains what is optional vs required
- [ ] No secrets
