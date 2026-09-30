# AI Stack Forge

Collaborative AI engineering lab that composes the 2026 open-source AI stack into one system a team can actually run together.

Source requirements: [Resnur AI — 15 AI GitHub Repositories You Should Explore in 2026](https://www.linkedin.com/posts/artificialintelligence-generativeai-machinelearning-share-7510891733761609729-CO8R).

**Repo:** https://github.com/ahmaddroobi99/ai-stack-forge

## What we are building together

A **team AI workspace** — not a tutorial dump.

Collaborators share one control plane that can:

- chat with local and served models
- retrieve from a shared knowledge base
- run agents and visual workflows
- generate images
- automate recurring jobs
- train / fine-tune models when needed
- expose the same assistant on Slack, Discord, Telegram, and the web

The 15 repositories in `REQUIREMENTS.md` are the **approved building blocks**. This repo is the glue, contracts, compose files, and design.

## Learning map (from the source post)

```
Train models        → PyTorch / TensorFlow
Use models          → Transformers
Run locally         → Ollama / llama.cpp
Serve LLMs          → vLLM
Build RAG           → RAGFlow
Build agents        → LangChain / Langflow / Flowise / Dify
Automate            → n8n
Generate images     → Stable Diffusion WebUI
Build AI interfaces → Open WebUI
Channel assistant   → OpenClaw
```

## Documents

| File | Purpose |
| --- | --- |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Full source post + resolved GitHub links |
| [docs/SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md) | Target architecture for the collaboration |
| [docs/STACK.md](docs/STACK.md) | How each repo maps to a service |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to pick a lane and ship |

## Suggested first slice (MVP)

1. Local inference: Ollama + Open WebUI
2. Shared RAG: RAGFlow over a team docs volume
3. One agent path: LangChain service behind an API
4. One automation: n8n webhook → RAG → model → Slack/Discord via OpenClaw
5. Optional image lane: Stable Diffusion WebUI

Do not stand up all 15 tools on day one. Compose them behind one gateway.

## Quick start (design repo)

```bash
git clone https://github.com/ahmaddroobi99/ai-stack-forge.git
cd ai-stack-forge
cp .env.example .env
# Read docs/SYSTEM_DESIGN.md before adding services
```

Compose profiles land in later PRs. Hardware reality: vLLM and Stable Diffusion want a GPU; Ollama / llama.cpp can start on CPU.

## Collaboration lanes

| Lane | Owner focus | Primary repos |
| --- | --- | --- |
| Inference | model serving, routing, quotas | vLLM, Ollama, llama.cpp |
| Knowledge | ingest, chunk, retrieve, cite | RAGFlow, Transformers |
| Agents | tools, memory, evals | LangChain, Dify, Langflow, Flowise |
| Interfaces | web + chat channels | Open WebUI, OpenClaw |
| Automation | triggers and human-in-the-loop | n8n |
| Media | image generation jobs | Stable Diffusion WebUI |
| Training | experiment jobs, checkpoints | PyTorch, TensorFlow |

## Status

Scaffold + requirements + system design. Implementation PRs should follow the contracts in `docs/SYSTEM_DESIGN.md`.

## License

MIT for original material in this repository. Upstream projects keep their own licenses — see `REQUIREMENTS.md`.
