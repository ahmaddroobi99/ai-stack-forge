# System design — AI Stack Forge

## 1. Problem

A team that wants to learn and ship with the 2026 open-source AI stack usually ends up with 15 tabs and no shared contracts. Knowledge lives in one person's chat history. Agents cannot see the same documents. Automations cannot call the same model router. Channels and the web UI diverge.

Forge is the collaboration layer: one workspace, one identity, one knowledge plane, one inference router, many optional studios.

## 2. Design principles

1. **Compose, do not rewrite.** Upstream projects stay upstream. We add adapters and a gateway.
2. **One request path.** Every user-visible action enters through the Forge gateway or OpenClaw, then hits the same router.
3. **Local-first.** Data, embeddings, and default models stay on team hardware.
4. **Lane ownership.** Collaborators own a plane, not a random file.
5. **Degrade gracefully.** If vLLM or SD WebUI is down, chat and RAG still work on Ollama.
6. **Cite and log.** RAG answers carry sources. Agent tool calls are auditable.

## 3. Target product

A shared lab where a collaborator can:

- ask a question in Open WebUI or Slack/Telegram via OpenClaw
- retrieve from the team knowledge base (RAGFlow)
- invoke a LangChain / Dify agent with tools
- trigger n8n for scheduled or webhook work
- enqueue an image job to Stable Diffusion WebUI
- launch a training job later (PyTorch / TensorFlow) without changing the chat contract

## 4. Logical architecture

```text
                     +------------------+
                     |  Chat channels   |
                     | Slack, Telegram, |
                     | Discord, iMessage|
                     +--------+---------+
                              |
                              v
+-------------+       +-------+--------+       +----------------+
| Open WebUI  |       | OpenClaw       |       | n8n / webhooks |
| (web chat)  |       | gateway        |       |                |
+------+------+       +-------+--------+       +--------+-------+
       |                      |                         |
       +----------+-----------+------------+------------+
                  |
                  v
         +--------+---------+
         |  Forge Gateway   |
         |  auth, route,    |
         |  quotas, audit   |
         +--+-----+-----+---+
            |     |     |
            |     |     +-----------------------------+
            |     |                                   |
            v     v                                   v
   +--------+--+ +--------+-------+          +--------+-------+
   | Inference | | Knowledge      |          | Workload plane |
   | Router    | | plane          |          |                |
   +-----+-----+ +--------+-------+          +--------+-------+
         |                |                           |
         |                v                           |
         |         +------+------+                    |
         |         | RAGFlow     |                    |
         |         | parsers,    |                    |
         |         | index, cite |                    |
         |         +-------------+
         |
         +--> vLLM  (GPU serve)
         +--> Ollama (local default)
         +--> llama.cpp (GGUF / edge)

Workload plane:
  LangChain API  |  Langflow  |  Dify  |  Flowise
  SD WebUI jobs  |  PyTorch / TensorFlow trainers
```

## 5. Component contracts

### 5.1 Forge gateway

Small service owned by this repo.

Responsibilities:

- authenticate team members (SSO later; shared token for MVP)
- normalize chat, tool, and job requests to an internal schema
- pick inference backend
- attach `trace_id`, user, workspace, model, token counts
- enforce max tokens / max images / max ingest size

MVP API (OpenAI-compatible subset):

- `POST /v1/chat/completions`
- `POST /v1/embeddings` (proxy to RAGFlow or a local embedding model)
- `POST /v1/jobs` (`rag_ingest` | `image_generate` | `train` | `workflow_run`)
- `GET  /v1/jobs/{id}`
- `GET  /v1/health`

### 5.2 Inference router

Policy:

```text
if model in vllm_registry and vllm.healthy: use vLLM
elif model in ollama_list: use Ollama
elif gguf_available: use llama.cpp
else: 503 with suggested pull command
```

All backends must look like an OpenAI chat API to the gateway. vLLM and Ollama already do. llama.cpp server mode does too.

### 5.3 Knowledge plane

RAGFlow is the system of record for documents.

Ingest path:

1. File or URL lands in `data/inbox/` or via `POST /v1/jobs` `rag_ingest`
2. n8n or the gateway calls RAGFlow ingest
3. RAGFlow parses, chunks, embeds, indexes
4. Chat requests with `use_rag=true` call RAGFlow retrieve, then the router generates with citations stuffed or tool-called

Citation rule: the UI must show source title + chunk locator. If RAGFlow is down, the gateway answers without retrieval and flags `rag=unavailable`.

### 5.4 Agent plane

- **Code agents:** LangChain service. Tools: `search_knowledge`, `run_workflow`, `generate_image`, `http_get` (allowlisted).
- **Visual graphs:** Langflow exports or publishes a webhook that the gateway can call.
- **Published apps:** Dify optional.
- **OpenClaw:** not a second agent runtime. It is the **channel adapter + personal/team assistant shell** that calls the gateway.

Memory:

- short-term: conversation store in Open WebUI / OpenClaw
- long-term team facts: RAGFlow only
- do not let each builder keep a private vector DB for team docs

### 5.5 Automation plane

N8n owns schedules, SaaS connectors, and human approval nodes.

Example flows:

- GitHub PR opened → summarize diff → post to Discord via OpenClaw
- New PDF in Drive → ingest to RAGFlow
- Nightly eval suite → write report to knowledge base

N8n never talks to models except through the gateway.

### 5.6 Media plane

Stable Diffusion WebUI runs as a worker with an API. Gateway enqueues `image_generate` jobs. Results land in object storage / local `data/artifacts/` and a URL is returned to chat.

### 5.7 Training plane (phase 2)

PyTorch is default. TensorFlow is supported for existing notebooks. Transformers is the model I/O layer. Jobs are batch, not interactive chat. Checkpoints go to `data/checkpoints/`. Successful adapters can be registered in the inference router (LoRA on vLLM, or a new Ollama model).

## 6. Data flow — answered question

```text
User message
  → Open WebUI or OpenClaw
  → Forge gateway (auth + trace_id)
  → optional RAGFlow retrieve (top_k, filters)
  → inference router
  → tokens stream back
  → audit log (prompt hash, sources, model, latency, tokens)
```

## 7. Deployment topology

### Lab laptop / single workstation

Profiles: `core` = Open WebUI + Ollama + RAGFlow + gateway + n8n.

### Team box (one GPU server)

Add vLLM + SD WebUI + OpenClaw. Builders (Langflow/Dify) can stay on a CPU VM.

### Split

- GPU node: vLLM, SD WebUI, optional training
- CPU node: RAGFlow, n8n, OpenClaw, builders, gateway

Network: private docker network. Only gateway, Open WebUI, OpenClaw, and n8n editor are published.

## 8. Security

- No public inbound to vLLM, Ollama, or RAGFlow admin.
- Secrets in `.env` / a local secret file, never committed.
- OpenClaw stays on team hardware; channel tokens are gateway-adjacent secrets.
- Tool `http_get` allowlist only.
- Image and ingest size caps.
- Prompt + document retention policy per workspace (default: 30 days for raw prompts, unlimited for approved knowledge docs).

## 9. Observability

Minimum:

- structured JSON logs with `trace_id`
- gateway `/v1/health` aggregates upstream health
- per-request: model, backend, tokens, retrieve_ms, generate_ms, error class

Later: OpenTelemetry traces into one collector.

## 10. Collaboration model

| Lane | Owns | Does not own |
| --- | --- | --- |
| Gateway | contracts, router, auth | model weights |
| Inference | vLLM / Ollama / llama.cpp deploy | prompt policy |
| Knowledge | RAGFlow pipelines, parsers | chat UI |
| Agents | LangChain tools, evals | channel tokens |
| Interfaces | Open WebUI theme, OpenClaw skills | index schema |
| Automation | n8n flows | inference policy |
| Media | SD workers | knowledge base |
| Training | job templates | production serving |

PRs must name the lane and the contract they touch.

## 11. Phased delivery

### Phase 0 — this repo
Requirements, design, compose skeleton.

### Phase 1 — Core lab
Gateway + Ollama + Open WebUI + RAGFlow + health checks.

### Phase 2 — Team channels
OpenClaw + n8n + one LangChain agent with `search_knowledge`.

### Phase 3 — Capacity
vLLM route, quotas, better evals.

### Phase 4 — Media + training
SD jobs and a single fine-tune template.

### Phase 5 — Studios
Langflow as builder of record. Dify optional. Flowise only if needed.

## 12. Risks

| Risk | Mitigation |
| --- | --- |
| Overlap of Langflow / Flowise / Dify / LangChain | one builder of record |
| GPU contention between vLLM, SD, training | job queue + exclusive lock |
| RAG quality | start with a small curated corpus and eval questions |
| Secret sprawl across 15 apps | gateway is the only secret consumer for model keys |
| Upstream breaking changes | pin image tags; adapters only |
| Scope explosion | phases above are hard gates |

## 13. Success metrics

- Time from clone to first cited RAG answer < 1 hour on a laptop with Ollama
- One shared knowledge corpus used by web UI, OpenClaw, and n8n
- 100% of model calls have a `trace_id`
- A new collaborator can pick a lane from CONTRIBUTING.md and open a PR without standing up all 15 tools
