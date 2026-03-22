---
project: MODULAR-RAG-MCP-SERVER
source_path: examples/MODULAR-RAG-MCP-SERVER
status: partially live-validated
confidence: medium
last_updated: 2026-03-22
next_action: compare its file-backed modular RAG and MCP pattern against LightRAG and deep-rag; only revisit runtime if a clean Chroma-backed query is needed
---

# MODULAR-RAG-MCP-SERVER Architecture

## Focus

This note captures the deeper static architecture findings that sit underneath the project summary in [`README.md`](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md).

## Observed Facts

- [`src/core/types.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/types.py) centralizes the core pipeline contracts for documents, chunks, processed chunk records, and retrieval-facing payloads. The document and chunk-family types enforce `source_path` in metadata, which gives the ingestion and retrieval layers a shared traceability contract.
- [`src/core/settings.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/settings.py) resolves repo-relative paths from the file location rather than the current working directory and validates config into immutable dataclasses. This makes settings and file locations intentionally CWD-independent.
- [`src/ingestion/pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/ingestion/pipeline.py) implements a staged ingestion pipeline, while [`src/core/query_engine/hybrid_search.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/hybrid_search.py) orchestrates query processing, dense retrieval, sparse retrieval, and fusion through injected components rather than hard-wired globals.
- [`src/core/query_engine/dense_retriever.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/query_engine/dense_retriever.py) is a narrow semantic retrieval component with explicit dependency injection for the embedding client and vector store, input validation, and normalized `RetrievalResult` output. The same design style appears across the query stack.
- [`src/core/trace/trace_context.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_context.py) and [`src/core/trace/trace_collector.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/core/trace/trace_collector.py) implement request-scoped trace collection with per-stage timing and JSONL persistence under a repo-resolved logs path.
- [`src/mcp_server/tools/query_knowledge_hub.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/query_knowledge_hub.py) shows the real runtime composition boundary:
  it caches the embedding client and reranker, but rebuilds collection-scoped vector-store and retriever objects so filesystem-backed collection data stays fresh across queries.
- That same tool also encodes the main storage assumptions directly:
  BM25 indexes are read from repo-local directories, vector data is collection-scoped, and query traces are persisted as files rather than held in memory.
- [`src/mcp_server/tools/list_collections.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/list_collections.py) and [`src/mcp_server/tools/get_document_summary.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/tools/get_document_summary.py) still bypass the vector-store abstraction and talk to Chroma directly.
- [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py) and the `mcp-server` script entry in [`pyproject.toml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/pyproject.toml) still point to the placeholder launcher rather than the real MCP server in [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py).

## Inferences

- The project's modularity is strongest inside the ingestion and retrieval core, where contracts, factories, and injected components are consistently applied.
- The MCP layer is real, but the system's operational state is still mostly file-backed rather than service-backed:
  config, Chroma data, BM25 indexes, and traces all live on disk and are re-opened or re-derived per query.
- This makes the implementation fairly restart-friendly, but also makes the project feel bootstrap-fragile:
  the codebase assumes the right files, Python packages, and path layout are already present.
- The main abstraction leak is not in the retrieval core but at the tool edges, where some MCP tools are still implicitly Chroma-only even though the repository advertises a pluggable vector-store layer.
- The project is best understood as an implementation-rich modular RAG system whose outer packaging and bootstrap ergonomics lag behind its internal architecture.

## Open Questions

- If the packaged entrypoint is corrected and the missing runtime dependencies are installed, does the real stdio server cleanly reach a successful non-error `query_knowledge_hub` response against a populated collection?
- How much of the intended observability experience is already usable from the existing trace JSONL output, and how much still depends on unfinished dashboard glue?
- Are the Chroma-specific MCP tools meant to remain provider-specific, or are they just ahead-of-abstraction shortcuts?

## Improvement Ideas

- Repoint the packaged `mcp-server` entry from [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py) to the real MCP server in [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py).
- Treat runtime bootstrap as a first-class product surface:
  document the dependency tiers needed for protocol-only, query-path, and full end-to-end validation.
- Either route `list_collections` and `get_document_summary` through the vector-store abstraction or document them as intentionally Chroma-bound.
- Reconcile config defaults and integration-test assumptions so runtime health signals are easier to trust.
