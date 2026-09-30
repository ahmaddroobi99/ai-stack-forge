# Stack mapping

How each required repository becomes a service in AI Stack Forge.

## Runtime planes

| Plane | Default | Scale-up | Fallback |
| --- | --- | --- | --- |
| Chat UI | Open WebUI | same | Dify web app |
| Channel assistant | OpenClaw gateway | team deploy of OpenClaw | n8n + webhooks |
| Local models | Ollama | llama.cpp for GGUF / edge | CPU Ollama |
| Served models | vLLM OpenAI-compatible API | multi-GPU vLLM | Ollama |
| RAG | RAGFlow | RAGFlow + extra parsers | LangChain retriever only |
| Programmatic agents | LangChain service | same | Dify API |
| Visual builders | Langflow **or** Flowise (pick one per team) | Dify studio | none |
| Automation | n8n | n8n queue mode | none |
| Images | Stable Diffusion WebUI API | dedicated GPU worker | disable profile |
| Training | PyTorch jobs | TensorFlow jobs | run offline |
| Model library | Transformers + Hub cache | private registry | local GGUF only |

## Conflict rules

Visual builders overlap. Do not run Langflow, Flowise, and Dify as three sources of truth.

- **Builder of record:** Langflow for graphs that export to code.
- **Product studio (optional):** Dify when non-engineers need to publish an app.
- **Flowise:** optional alternate; enable only if a collaborator already owns flows there.

Inference overlap:

- Open WebUI talks to an **inference router**, never to three runtimes at once from the UI.
- Router policy: `vLLM if model is registered and GPU is free, else Ollama, else llama.cpp`.

## Ports (local lab)

| Service | Host port |
| --- | --- |
| Open WebUI | 3000 |
| OpenClaw control UI | 3010 |
| RAGFlow | 80 / 9380 |
| Langflow | 7860 |
| Flowise | 3001 |
| Dify | 3002 |
| n8n | 5678 |
| vLLM | 8000 |
| Ollama | 11434 |
| SD WebUI | 7861 |
| Forge gateway | 8080 |

Adjust in `.env`. Do not hard-code ports in application code.
