---
project: LightRAG
source_path: examples/LightRAG
status: partially live-validated
confidence: medium
last_updated: 2026-03-22
next_action: compare its graph-plus-vector retrieval topology against deep-rag and MODULAR-RAG-MCP-SERVER; only revisit server or WebUI if serving-layer evidence is needed
---

# LightRAG Architecture

## Focus

This note captures the deeper static architecture findings that sit underneath the project summary in [`README.md`](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md).

## Observed Facts

- [`lightrag/lightrag.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) uses a single large `LightRAG` dataclass as the main control plane. It owns backend selection, workspace choice, chunking and extraction limits, token budgets, concurrency knobs, and retrieval defaults.
- The same class initializes a multi-store runtime rather than a single index:
  document KV stores, chunk KV stores, entity and relation stores, graph storage, three vector stores, LLM cache, and document-status storage are all separate components.
- [`initialize_storages()`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) first reconciles the default workspace, then auto-initializes workspace-scoped `pipeline_status`, and only then initializes the concrete storages. This means runtime readiness is tied to storage fabric setup, not just object construction.
- [`lightrag/kg/shared_storage.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/kg/shared_storage.py) is not a small helper. It implements shared namespace data, workspace defaults, keyed locks, and multi-process-aware lock cleanup. It is effectively a runtime state substrate for the rest of the system.
- That same shared-storage layer also materializes a workspace-scoped `pipeline_status` record with fields such as `busy`, `job_name`, `latest_message`, and `history_messages`, so operational state is explicit and shared rather than hidden in request-local objects.
- [`ainsert(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) is a pipeline entrypoint, not a thin upsert helper:
  it enqueues documents, processes the queue, and relies on document-status tracking and storage lifecycle rules.
- [`aquery_data(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) and [`aquery_llm(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) share the same retrieval dispatch and differ mainly in whether generation is performed after retrieval.
- Query-mode boundaries are encoded centrally:
  `local`, `global`, `hybrid`, and `mix` all route through [`kg_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py), while `naive` routes through [`naive_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py), and `bypass` skips retrieval.
- [`kg_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) derives high-level and low-level keywords, falls back to using the raw query as low-level keywords for short queries when both keyword lists are empty, builds retrieval context, and only then decides whether to return context, prompt, or generated output.
- [`_perform_kg_search(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) pre-computes embeddings in one batch for the active retrieval branches and, in `mix` mode, adds vector chunk retrieval on top of KG-side entity and relation retrieval.
- [`naive_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) is architecturally narrower:
  it pulls only chunk vectors, computes a dynamic chunk token budget after prompt and query overhead, then builds references and optional generation from that chunk set.
- [`api/routers/query_routes.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/routers/query_routes.py) exposes the retrieval system almost one-to-one:
  mode choice, rerank toggle, context-only or prompt-only mode, history injection, streaming, and reference options are all API-level switches.
- [`api/lightrag_server.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/lightrag_server.py) wraps the core with lifespan-managed storage initialization and finalization, auth, WebUI serving, and Ollama-compatible API surfaces, so the serving layer is materially larger than a thin REST wrapper.

## Inferences

- `LightRAG` is centralized at the control-plane level but distributed at the state level:
  one dataclass orchestrates the system, while actual runtime state is spread across many stores plus the shared-storage substrate.
- The project is not just graph-augmented retrieval. It is also an operational runtime for ingestion, shared status, and multi-process serving.
- The explicit separation between `conversation_history` and retrieval inputs indicates that LightRAG treats retrieval as a structured substrate problem, not as a conversational-memory problem.
- `mix` is the most representative mode because it expresses the platform's intended identity:
  KG-aware retrieval plus direct vector chunk retrieval, with the chunk branch still able to carry useful results when KG extraction is sparse.
- The system's flexibility comes with substantial operational complexity:
  workspace defaults, shared locks, many backends, and many serving modes all widen the possible failure surface.

## Open Questions

- How much extra behavior appears once requests pass through the full FastAPI and WebUI layers rather than the already-validated object-level paths?
- Which storage and provider combinations are actually exercised regularly, versus merely supported by the abstraction surface?
- How much of answer quality in real deployments comes from KG retrieval, chunk retrieval, reranking, or prompt shaping?

## Improvement Ideas

- Document the mode boundaries and fallback rules more explicitly, especially the practical difference between `hybrid`, `mix`, and `naive`.
- Separate the "smallest working path" from the "maximum flexibility path" more clearly in docs and examples, so new users do not need to understand the whole backend matrix to get started.
- Keep using object-level and mode-level validation as the default research entrypoint, because it isolates the retrieval substrate from the much larger server and deployment surface.
