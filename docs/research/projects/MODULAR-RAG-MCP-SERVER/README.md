---
project: MODULAR-RAG-MCP-SERVER
source_path: examples/MODULAR-RAG-MCP-SERVER
status: partially live-validated
confidence: medium
last_updated: 2026-03-22
next_action: compare its file-backed modular RAG and MCP pattern against LightRAG and deep-rag; only revisit runtime if a clean Chroma-backed query is needed
---

# MODULAR-RAG-MCP-SERVER

## Project Summary

`MODULAR-RAG-MCP-SERVER` is no longer just a low-confidence GitHub observation.

With the local checkout in place, it now reads as a real modular RAG codebase with three distinct layers:

- a spec- and Skill-heavy teaching shell
- a substantial ingestion and retrieval implementation
- an MCP exposure layer that is partly real but still not aligned with the default packaging and environment state

## Quick Classification

- Style: modular RAG platform with MCP packaging and teaching-oriented workflow docs
- Best fit: a learning-oriented but genuinely implemented RAG engineering project, not just a paper scaffold
- Current confidence boundary: medium; the static analysis is now complete for the local checkout, and the remaining uncertainty is mostly in optional deeper runtime validation rather than missing architectural coverage

## Key Files

- [`README.md`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/README.md)
- [`DEV_SPEC.md`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/DEV_SPEC.md)
- [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py)
- [`pyproject.toml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/pyproject.toml)
- [`config/settings.yaml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/config/settings.yaml)
- [`src/core/settings.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/settings.py)
- [`src/ingestion/pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/ingestion/pipeline.py)
- [`src/core/query_engine/query_processor.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/query_processor.py)
- [`src/core/query_engine/hybrid_search.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/hybrid_search.py)
- [`src/core/types.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/types.py)
- [`src/core/trace/trace_context.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_context.py)
- [`src/core/trace/trace_collector.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_collector.py)
- [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py)
- [`src/mcp_server/protocol_handler.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/protocol_handler.py)
- [`src/mcp_server/tools/query_knowledge_hub.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/query_knowledge_hub.py)
- [`src/mcp_server/tools/list_collections.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/list_collections.py)
- [`src/mcp_server/tools/get_document_summary.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/get_document_summary.py)
- [`tests/integration/test_mcp_server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/tests/integration/test_mcp_server.py)
- [`tests/integration/test_ingestion_pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/tests/integration/test_ingestion_pipeline.py)
- [`architecture.md`](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/architecture.md)

## Observed Facts

- The project is now present locally under [`examples/MODULAR-RAG-MCP-SERVER`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER), so this round is based on local source inspection rather than only GitHub reading.
- The root launcher in [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py) still behaves like a placeholder:
  it loads settings, initializes logging, prints a startup banner, logs `MCP Server will be implemented in Phase E.`, and exits `0`.
- The package script in [`pyproject.toml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/pyproject.toml) still points to `mcp-server = "main:main"`, so the packaged default entrypoint routes through that placeholder path.
- At the same time, [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py) contains a real stdio MCP server implementation using the official Python MCP SDK.
- [`src/mcp_server/protocol_handler.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/protocol_handler.py) registers real tools and exposes `tools/list` plus `tools/call` handlers through the low-level MCP server.
- The default tool set includes:
  - `query_knowledge_hub`
  - `list_collections`
  - `get_document_summary`
- A local smoke run of `python3 examples/MODULAR-RAG-MCP-SERVER/main.py` succeeded and confirmed that the root entrypoint is still just the placeholder path.
- In an isolated local venv under [`examples/MODULAR-RAG-MCP-SERVER/.venv-research`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/.venv-research), installing `mcp` alone was not enough:
  the real server path still failed until `pyyaml` was also installed.
- After installing `mcp` and `pyyaml` in that venv, a real stdio MCP handshake against [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py) succeeded for:
  - `initialize`
  - `notifications/initialized`
  - `tools/list`
- That real `tools/list` response advertised all three expected tools:
  `query_knowledge_hub`, `list_collections`, and `get_document_summary`.
- A real stdio `tools/call query_knowledge_hub` also succeeded at the protocol level.
  The server returned a valid `CallToolResult` with `isError: true` rather than crashing or breaking JSON-RPC.
- That same real `tools/call` path surfaced the dependency ladder clearly:
  - before installing `jieba`, the tool failed during query-processor import
  - after installing `jieba`, it progressed further and then failed at the Chroma layer because `chromadb` was still missing
- [`src/ingestion/pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/ingestion/pipeline.py) implements a full six-stage ingestion pipeline:
  integrity check, document loading, chunking, transform, encoding, and storage.
- [`src/core/query_engine/hybrid_search.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/hybrid_search.py) is not stub code.
  It performs query preprocessing, parallel dense and sparse retrieval, RRF fusion, and fallback behavior.
- [`src/core/types.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/types.py), [`src/core/settings.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/settings.py), [`src/core/trace/trace_context.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_context.py), and [`src/core/trace/trace_collector.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_collector.py) show that the project has a real internal control plane for typed contracts, repo-anchored path resolution, and file-backed per-request tracing.
- [`src/core/query_engine/query_processor.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/query_processor.py) implements bilingual keyword extraction and `key:value` filter parsing for retrieval.
- [`src/mcp_server/tools/query_knowledge_hub.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/query_knowledge_hub.py) rebuilds collection-specific retriever state per query, uses the modular query engine, and records query traces.
- The same tool caches only some components:
  embedding client and reranker are reused, while collection-specific vector store and retrievers are rebuilt.
- A minimal local invocation of [`QueryKnowledgeHubTool.execute(...)`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/query_knowledge_hub.py) also succeeded when I injected:
  - a fake `HybridSearch`
  - a fake `ResponseBuilder`
  - a no-op `_ensure_initialized(...)`
  This confirmed that the tool's async execution path, trace creation, and response plumbing work independently of the heavier storage/provider initialization.
- [`src/mcp_server/tools/list_collections.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/list_collections.py) and [`src/mcp_server/tools/get_document_summary.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/get_document_summary.py) do not go through the vector-store abstraction.
  Both open Chroma directly from the configured persist directory.
- [`src/libs/vector_store/vector_store_factory.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/libs/vector_store/vector_store_factory.py) advertises pluggable vector stores, but the local vector-store package currently only contains [`chroma_store.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/libs/vector_store/chroma_store.py), and [`src/libs/vector_store/__init__.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/libs/vector_store/__init__.py) only auto-registers `chroma`.
- [`config/settings.yaml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/config/settings.yaml) currently defaults `llm.provider` and `embedding.provider` to `openai`.
- [`tests/integration/test_ingestion_pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/tests/integration/test_ingestion_pipeline.py) still asserts Azure-oriented expectations such as `settings.llm.provider == "azure"` and `settings.embedding.provider == "azure"`, which no longer match the checked-in default config.
- The test tree is broad and covers:
  - protocol handling
  - MCP server initialization
  - ingestion pipeline
  - hybrid search
  - reranking
  - evaluation
  - observability
  - vector-store contracts
- I could not execute the pytest suite in this workspace because `python3 -m pytest ...` failed with `No module named pytest`.

## Inferences

- This repository is more than a teaching shell.
  The ingestion, retrieval, MCP, and observability layers are materially implemented.
- The most important drift is no longer "README versus no code".
  It is now "real code versus default launch and environment health."
- The modularity story is only partially realized today.
  The core query path is abstracted, but some MCP tools are still Chroma-specific, and the vector-store layer currently has only one concrete implementation in-tree.
- The project currently looks like a real modular RAG system whose packaging and bootstrap path lag behind the internal implementation.
- The MCP layer is now past the "paper implementation" threshold.
  The real server can initialize and enumerate tools once the minimal runtime deps are present.
- The live `tools/call` result suggests the MCP surface is robust even when backend dependencies are incomplete:
  failures are being converted into tool-level error responses instead of protocol-level crashes.
- The deeper static pass sharpens the state model:
  most operational context is externalized to repo-local files and directories such as config, Chroma data, BM25 indexes, and JSONL traces, rather than held in long-lived MCP server memory.
- The strongest modularity is in the ingestion and retrieval core, while the biggest abstraction leak is at the tool edge where some MCP tools still assume Chroma directly.
- This makes it a stronger comparison target than I first judged from the remote-only pass, but still not a clean end-to-end sample until the dependency and launcher story is reconciled.

## Open Questions

- After installing broader declared dependencies, does `tools/call` for `query_knowledge_hub` succeed over the real stdio server without fake search components?
- After installing `chromadb` and the configured embedding-provider stack, does the real stdio `tools/call query_knowledge_hub` return a non-error retrieval response rather than a dependency error?
- How many of the integration tests are healthy versus stale under the current `settings.yaml` defaults?
- Is the Chroma-specific logic in `list_collections` and `get_document_summary` intentional, or just an early implementation shortcut?
- How complete is the dashboard and observability experience in practice, compared with the ambitious README and DEV_SPEC framing?
- Which parts of the modular backends are truly interchangeable today, and which parts are still effectively single-provider?

## Improvement Ideas

- Repoint the packaged `mcp-server` script away from [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py) and toward the real MCP server entrypoint in [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py).
- Reconcile [`config/settings.yaml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/config/settings.yaml) with the assumptions baked into [`tests/integration/test_ingestion_pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/tests/integration/test_ingestion_pipeline.py).
- Either generalize `list_collections` and `get_document_summary` through the vector-store abstraction or explicitly document that those tools are Chroma-only today.
- Add one documented bootstrap path that installs runtime and test dependencies together, so research and smoke validation do not stop at missing `mcp`, `pyyaml`, `chromadb`, or `pytest`.
- Keep the deeper architectural note in [`architecture.md`](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/architecture.md) as the static reference point, so the project README can stay focused on status, evidence, and comparison-ready conclusions.
