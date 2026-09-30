# Requirements

This file is the source-of-truth requirements captured from the LinkedIn post, plus resolved repository links.

## Source

- Author: Resnur AI
- Title: 15 AI GitHub Repositories You Should Explore in 2026
- Post: https://www.linkedin.com/posts/artificialintelligence-generativeai-machinelearning-share-7510891733761609729-CO8R
- Short URL provided: https://lnkd.in/p/eWnMDgzh
- Image credit (original post): Rathnakumar Udayakumar

## Original post text

> 🚀 15 AI GitHub Repositories You Should Explore in 2026
>
> If you're learning AI, LLMs, RAG, AI agents, or generative AI, don't just watch tutorials.
> Read the code. Build with it. Break it. Learn from it.
>
> These 15 open-source repositories cover a huge part of the modern AI stack — from training deep learning models to running LLMs locally and building production-ready AI applications.
>
> Here's the list
>
> 1. Stable Diffusion WebUI — Image generation & experimentation
> 2. vLLM — High-performance LLM inference & serving
> 3. RAGFlow — RAG & document intelligence
> 4. Langflow — Visual AI & LLM workflow building
> 5. TensorFlow — Deep learning & ML development
> 6. PyTorch — Research, deep learning & model development
> 7. n8n — Workflow automation & AI integrations
> 8. Transformers — Pretrained models & NLP/vision/audio
> 9. Ollama — Run LLMs locally
> 10. llama.cpp — Efficient local LLM inference
> 11. Dify — Build AI applications & agent workflows
> 12. LangChain — LLM applications, agents & orchestration
> 13. Open WebUI — Local AI interfaces & model interaction
> 14. Flowise — Visual LLM & agent workflow builder
> 15. OpenClaw — Explore open-source AI workflows & tooling
>
> Think of this as an AI learning map:
>
> Train models → PyTorch / TensorFlow
> Use models → Transformers
> Run locally → Ollama / llama.cpp
> Serve LLMs → vLLM
> Build RAG → RAGFlow
> Build agents & workflows → LangChain / Langflow / Flowise / Dify
> Automate → n8n
> Generate images → Stable Diffusion WebUI
> Build AI interfaces → Open WebUI
>
> You don't need to study all 15 at once.
> Pick one → clone it → read the architecture → run it locally → modify something → build with it.
> That's where the real learning starts.

## Project requirements derived from the post

The collaboration does **not** re-implement these projects. It **composes** them.

1. The approved stack is the 15 repositories below. New dependencies need a short ADR.
2. Learning mode is mandatory: each service lane must document how a request flows through that upstream project.
3. Local-first is the default. Cloud APIs are adapters, not the core.
4. The product is a shared workspace a team can run, not 15 disconnected demos.
5. Start with one repo per lane, then add the others as optional profiles.
6. Observability, failure handling, and citations matter as much as model quality.

## Resolved repository links

| # | Project | Role in this collaboration | GitHub |
| --- | --- | --- | --- |
| 1 | Stable Diffusion WebUI | Image generation jobs | https://github.com/AUTOMATIC1111/stable-diffusion-webui |
| 2 | vLLM | High-performance LLM serving | https://github.com/vllm-project/vllm |
| 3 | RAGFlow | Shared document intelligence / RAG | https://github.com/infiniflow/ragflow |
| 4 | Langflow | Visual agent / LLM workflow builder | https://github.com/langflow-ai/langflow |
| 5 | TensorFlow | Training and classic ML jobs | https://github.com/tensorflow/tensorflow |
| 6 | PyTorch | Research training and fine-tunes | https://github.com/pytorch/pytorch |
| 7 | n8n | Automation and integrations | https://github.com/n8n-io/n8n |
| 8 | Transformers | Model I/O, tokenizers, pipelines | https://github.com/huggingface/transformers |
| 9 | Ollama | Default local model runtime | https://github.com/ollama/ollama |
| 10 | llama.cpp | Efficient local / edge inference | https://github.com/ggerganov/llama.cpp |
| 11 | Dify | App + agent studio | https://github.com/langgenius/dify |
| 12 | LangChain | Programmatic agents and tools | https://github.com/langchain-ai/langchain |
| 13 | Open WebUI | Primary web chat UI | https://github.com/open-webui/open-webui |
| 14 | Flowise | Alternate visual workflow builder | https://github.com/FlowiseAI/Flowise |
| 15 | OpenClaw | Local-first assistant across chat channels | https://github.com/openclaw/openclaw |

LinkedIn short links from the original post (kept for provenance):

1. https://lnkd.in/eVpdh933
2. https://lnkd.in/ey_fWtgB
3. https://lnkd.in/eyDNXApH
4. https://lnkd.in/erXFZHbp
5. https://lnkd.in/eCqrTgdT
6. https://lnkd.in/eXc-3erm
7. https://lnkd.in/ekEp_3Ma
8. https://lnkd.in/eZbxYTdZ
9. https://lnkd.in/esgD3VZm
10. https://lnkd.in/eYRKEty7
11. https://lnkd.in/en6AD2WX
12. https://lnkd.in/eYs9NKWA
13. https://lnkd.in/e7C5UwxX
14. https://lnkd.in/ejhK5T5B
15. https://lnkd.in/eTy64Dvw

## Non-goals

- Forking all 15 repos into this monorepo
- Replacing upstream UIs unless a contract requires it
- Training foundation models from scratch as an MVP goal
- Shipping a hosted SaaS in v0
