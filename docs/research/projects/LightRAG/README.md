---
project: LightRAG
source_path: examples/LightRAG
status: partially live-validated
confidence: medium
last_updated: 2026-03-22
next_action: compare its graph-plus-vector retrieval topology against `deep-rag` and `MODULAR-RAG-MCP-SERVER`; only revisit server or WebUI if serving-layer evidence is needed
---

# LightRAG

## Project Summary

`LightRAG` is not a small demo RAG wrapper.

It is a graph-augmented RAG platform that combines a core Python library, a FastAPI server, a WebUI, an Ollama-compatible API surface, a large storage abstraction layer, and an operational setup stack for local, Docker, and Kubernetes deployment.

## Quick Classification

- Style: graph-centric RAG platform with API/WebUI packaging
- Best fit: configurable knowledge-graph plus vector retrieval over document corpora
- Deep research behavior: low; it performs structured retrieval and context assembly, not agent-style multi-round planning
- Current confidence boundary: medium; the static architecture coverage is now complete for the local checkout, and the remaining uncertainty is mostly about how much extra behavior appears in the larger serving layer

## Key Files

- `examples/LightRAG/README.md`
- `examples/LightRAG/pyproject.toml`
- `examples/LightRAG/env.example`
- `examples/LightRAG/lightrag/lightrag.py`
- `examples/LightRAG/lightrag/operate.py`
- `examples/LightRAG/lightrag/base.py`
- `examples/LightRAG/lightrag/kg/shared_storage.py`
- `examples/LightRAG/lightrag/api/lightrag_server.py`
- `examples/LightRAG/lightrag/api/config.py`
- `examples/LightRAG/lightrag/api/routers/query_routes.py`
- [`architecture.md`](/Users/uynil/projects/agents/docs/research/projects/LightRAG/architecture.md)

## Observed Facts

- The repository is packaged as `lightrag-hku` in [`pyproject.toml`](/Users/uynil/projects/agents/examples/LightRAG/pyproject.toml) and exposes both core-library and server entrypoints.
- Optional dependency groups in [`pyproject.toml`](/Users/uynil/projects/agents/examples/LightRAG/pyproject.toml) include `api`, `offline-storage`, `offline-llm`, `evaluation`, and `observability`, which makes the project more like a platform distribution than a single narrow library.
- [`README.md`](/Users/uynil/projects/agents/examples/LightRAG/README.md) positions LightRAG as "simple and fast," but the actual repository includes:
  - core indexing and query code
  - a FastAPI server
  - a WebUI build
  - evaluation tooling
  - tracing support
  - multiple deployment paths
  - many storage backends
- The core `LightRAG` object in [`lightrag.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) is a large dataclass that owns:
  - storage backend selection
  - workspace isolation
  - chunking and extraction limits
  - token-budget controls
  - concurrency controls
  - graph and vector retrieval settings
- The `LightRAG` runtime is explicitly multi-store.
  [`lightrag.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) initializes separate stores for:
  - LLM response cache
  - full documents
  - text chunks
  - entity and relation records
  - graph storage
  - entity, relationship, and chunk vector indexes
  - document status
- [`initialize_storages()`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) first reconciles the default workspace and auto-initializes workspace-scoped `pipeline_status`, which means the runtime's readiness model depends on shared storage state, not just object construction.
- [`lightrag/kg/shared_storage.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/kg/shared_storage.py) implements workspace defaults, shared namespace data, keyed locks, and multi-process-safe lock cleanup. It is a real state-management layer, not just a few helper utilities.
- [`base.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/base.py) defines `QueryParam.mode` values:
  `local`, `global`, `hybrid`, `naive`, `mix`, and `bypass`.
- `QueryParam.mode` defaults to `mix`, not `naive` or `hybrid`.
- [`base.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/base.py) also makes the contract boundary explicit:
  `conversation_history` is for LLM response context, not retrieval.
- [`lightrag.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py) dispatches `kg_query(...)` for `local`, `global`, `hybrid`, and `mix`, and dispatches `naive_query(...)` for the other path.
- In [`lightrag.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/lightrag.py), `aquery_data(...)` forces `only_need_context=True` and returns structured retrieval output without generation, while `aquery_llm(...)` uses the same retrieval paths but adds LLM response generation.
- [`operate.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) contains the heavy retrieval and indexing logic, including:
  - chunking by token size
  - entity/relation summary merging
  - graph-aware query construction
  - naive vector retrieval
  - cache-aware LLM calls
  - reference generation
- [`kg_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) first derives high-level and low-level keywords, then builds graph-aware context through `_build_query_context(...)`, and finally either returns context/prompt directly or calls the LLM with cache-aware arguments.
- That same `kg_query(...)` path explicitly falls back to using the raw query as low-level keywords for short queries when both keyword lists are empty, which explains part of the project's tolerance for incomplete keyword extraction.
- [`naive_query(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) is the chunk-only path:
  it retrieves chunks from the chunk vector store, computes a token budget for chunk packing, builds references, and then optionally calls the LLM.
- [`_perform_kg_search(...)`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/operate.py) pre-computes embeddings for the active retrieval branches in one batch and, in `mix` mode, adds direct chunk-vector retrieval on top of KG-side entity and relation retrieval.
- The mode differences are architectural, not cosmetic:
  - `local`: entity-centric KG retrieval
  - `global`: relation-centric KG retrieval
  - `hybrid`: combines local and global KG retrieval
  - `mix`: combines KG retrieval with direct chunk vector retrieval
  - `naive`: direct chunk vector retrieval only
  - `bypass`: direct LLM path without retrieval
- [`query_routes.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/routers/query_routes.py) exposes API-level controls for:
  - query mode
  - context-only or prompt-only returns
  - response format
  - history injection
  - rerank toggling
  - references and chunk content inclusion
  - streaming vs non-streaming responses
- [`query_routes.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/routers/query_routes.py) defaults `include_references=True` at the API model level.
- [`lightrag_server.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/lightrag_server.py) positions the server as more than a thin API:
  it includes WebUI serving, auth, frontend build checks, router composition, and Ollama-emulation support.
- [`api/config.py`](/Users/uynil/projects/agents/examples/LightRAG/lightrag/api/config.py) exposes a very broad CLI/env configuration surface:
  host/port, working and input directories, auth, SSL, workers, storage, LLM bindings, embedding bindings, rerank bindings, token budgets, and workspace selection.
- [`env.example`](/Users/uynil/projects/agents/examples/LightRAG/env.example) is extensive and covers:
  - server/auth/SSL settings
  - query token budgets
  - reranker configuration
  - document processing limits
  - concurrency controls
  - multiple LLM and embedding backends
  - storage backends and workspace isolation
- The repository contains many storage implementations under `lightrag/kg/`, including local JSON/NetworkX defaults plus Redis, Neo4j, Memgraph, Milvus, MongoDB, PostgreSQL, Qdrant, OpenSearch, and others.
- [`README.md`](/Users/uynil/projects/agents/examples/LightRAG/README.md) explicitly describes four storage roles:
  KV storage, vector storage, graph storage, and document-status storage.
- [`README.md`](/Users/uynil/projects/agents/examples/LightRAG/README.md) also describes evaluation, reranking, graph visualization, document deletion, and manual entity/relation editing as first-class capabilities.
- In an isolated local venv under [`examples/LightRAG/.venv-research`](/Users/uynil/projects/agents/examples/LightRAG/.venv-research), I ran a direct offline `naive_query(...)` call with a fake chunk vector store.
  It returned:
  - a chunk-only context payload
  - `reference_count = 1`
  - naive-mode metadata showing `total_chunks_found = 2` and `final_chunks_count = 2`
- In that same venv, I also ran `_perform_kg_search(...)` in `mode=\"mix\"` with fake graph storage, fake entity/relation vector stores, and a fake chunk vector store.
  That path produced:
  - `entity_count = 2`
  - `relation_count = 1`
  - `vector_chunk_count = 1`
  - non-empty `chunk_tracking` for the vector chunk branch
- Those two runs confirm that the local `naive` and `mix` paths are behaviorally distinct in execution, not just in type signatures or docs.
- In that same venv, I also ran a real `LightRAG` object lifecycle with:
  - `initialize_storages()`
  - `ainsert(...)`
  - `aquery_data(..., mode=\"naive\")`
  That runtime path succeeded with `chunk_count = 1` and `reference_count = 1`.
- I then ran that same real object with `aquery_data(..., mode=\"mix\")`.
  Even though the dummy LLM produced no extracted entities or relations during ingestion, the query still returned:
  - `status = success`
  - `entity_count = 0`
  - `relation_count = 0`
  - `chunk_count = 1`
  - `reference_count = 1`
  which shows that `mix` can degrade to the chunk/vector branch while still following the mixed retrieval path.

## Inferences

- `LightRAG` is better understood as a graph-centric RAG platform than as a narrowly scoped retrieval library.
- The "light" in the name appears to refer more to retrieval design or usability goals than to implementation surface area.
- The core retrieval idea is not agentic search.
  It is structured query-time context assembly over a combination of graph data, vector data, and extracted keywords.
- Its most representative mode is probably `mix`, because that path expresses the project's actual product identity:
  graph-aware retrieval plus direct chunk retrieval, but still inside a retrieval substrate rather than an autonomous loop.
- The new offline runs support that interpretation:
  `naive` behaves like chunk-only retrieval, while `mix` really does combine KG-side and vector-chunk-side search signals.
- The real-object runs add a second useful conclusion:
  `mix` can still produce a successful answer path when KG extraction yields no entities, because the chunk branch continues to carry retrieval.
- The deeper static pass sharpens that conclusion:
  the control plane is centralized in one `LightRAG` object, but the runtime state is distributed across many stores plus a shared-storage substrate with workspace and lock semantics.
- This project sits in a different family from `deep-rag`.
  `deep-rag` gives the model a knowledge map and lets it navigate files directly; `LightRAG` precomputes and maintains a much heavier graph/vector substrate and then exposes mode-based retrieval over that substrate.
- It also sits in a different family from agentic research systems like `DeepSearchAgent-Demo`, `gpt-researcher`, and `autoresearch`.
  Those are orchestration systems; `LightRAG` is a retrieval platform and serving layer.
- The API/server layer is operationally significant enough that a meaningful analysis of LightRAG cannot stop at the core `LightRAG` class.
- The repository appears designed for broad deployment flexibility, but that also means the effective behavior may vary substantially across binding and storage combinations.

## Open Questions

- How different are `local`, `global`, `hybrid`, and `mix` in the actual prompt/context payload sent to the LLM?
- How much of LightRAG's answer quality comes from the graph layer versus the vector layer versus reranking?
- Is `mix` genuinely the best default mode across corpora, or mainly a good default when reranking is enabled?
- How expensive is the indexing and merge pipeline in practice compared with simpler file-navigation systems like `deep-rag`?
- Which operational paths are actually healthy today:
  core library only, API server, WebUI, Ollama-emulation, and the more exotic storage backends?
- How much extra behavior appears once the API server and WebUI layers are involved, beyond these already-validated object-level retrieval runs?
- How much state and failure complexity comes from supporting so many storage and provider combinations?

## Improvement Ideas

- Deep-dive `kg_query(...)` and `naive_query(...)` to understand the real context-packing difference between query modes.
- Inspect the prompt templates and keyword-extraction flow to map how query semantics are translated into retrieval behavior.
- Run one real server `/query` path, or one minimal WebUI flow, to validate how much additional behavior appears above these already-validated object-level retrieval paths.
- Compare LightRAG's graph-plus-vector design directly against `deep-rag`'s file-navigation design and `MODULAR-RAG-MCP-SERVER`'s modular pipeline design in the comparison docs after one more focused pass.
- Keep the deeper architectural note in [`architecture.md`](/Users/uynil/projects/agents/docs/research/projects/LightRAG/architecture.md) as the static reference point, so the project README can stay focused on status, evidence, and comparison-ready conclusions.
