# Local-Knowledge Context Management Patterns

## Comparison Goal

Compare how local-knowledge retrieval systems keep, retrieve, compress, and hand off context during one request and across repeated use.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- main working context
- durable evidence layer
- retrieval style
- context packing
- update rule
- forgetting behavior
- resumability shape

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Main working context | the current `data_structure.md` branch, grep hits, and format-specific local extracts | frontend-replayed message history, one global file-summary map, and current tool observations | request-scoped query objects, retrieval results, and trace context | mode-specific retrieval state assembled from graph, vector, KV, cache, and document-status stores |
| Durable evidence layer | the corpus itself, manual index docs, and extracted side artifacts like `.txt` files | knowledge-base files, mirrored markdown chunks, `summary.txt`, and browser-held chat history | repo-local config, collection data, BM25 indexes, vector-store data, and JSONL traces | workspace-scoped shared storage plus graph stores, vector stores, KV stores, caches, and document-status state |
| Retrieval style | index-guided manual narrowing before file-type-specific reading | model-directed path retrieval over a summarized knowledge map | typed retrieval pipeline with dense/sparse fusion and MCP tool exposure | mode-driven graph-plus-vector context assembly with optional direct chunk branch |
| Context packing | keep the active view small by reading only the relevant index, file slice, or worksheet subset | inject summary up front, then append raw file or directory payloads as tool results or ReAct observations | return structured retrieval results and traces instead of replaying a growing transcript | compute token-budgeted KG/chunk context and optionally separate context-only from answer generation |
| Update rule | switch index branch, refine keywords, or change file workflow; no shared runtime state object is updated | frontend resends full message history each turn; backend appends tool results or synthetic observations inside the loop | rebuild collection-scoped retrieval state per query and persist traces after each request | update many stores during ingestion; query-time context is assembled from persisted stores rather than one growing chat state |
| Forgetting behavior | anything not re-opened through indexes or file previews falls out of scope | little backend-side forgetting; long chat history and large retrieval payloads are replayed unless the client trims them | request-local context is dropped after response, with traces preserved separately | unused retrieval branches are trimmed by budgets and mode selection while durable stores preserve the broader substrate |
| Resumability shape | manual operational resume through the same corpus, indexes, and extracted helpers | weak server-side resume, moderate artifact resume, and strong dependence on frontend replay | strong artifact-level resume through collections and traces, but not a conversational session layer | strong substrate-level resume through shared stores and workspace state, but not a chat-centric memory model |

## Shared Patterns

- all four rely on durable external artifacts more than on rich in-memory agent memory
- all four make context management depend on what gets dropped, not only what gets preserved
- all four separate long-lived corpus state from short-lived request or loop state, but they do so with very different mechanisms

## Key Differences

- `rag-skill` manages context procedurally:
  the agent is supposed to stay narrow by obeying indexes and pre-read gates
- `deep-rag` manages context through replay plus raw observation injection:
  lightweight to implement, but prone to payload growth
- `MODULAR-RAG-MCP-SERVER` manages context as retrieval products and traces, not as a conversational working set
- `LightRAG` manages context through a persistent retrieval substrate that assembles budgeted context on demand rather than carrying one growing transcript

## Conclusions

These projects suggest four context-management patterns:

- procedural file-navigation context
- replay-plus-observation context
- request-scoped retrieval-and-trace context
- substrate-assembled multi-store context

That split is useful because these systems are not all solving "memory" in the same layer.

If the goal is lightweight local workflow discipline, `rag-skill` is the clearest reference.

If the goal is interactive retrieval over a file-backed corpus, `deep-rag` is the clearest reference.

If the goal is stable service-side retrieval context without chat-heavy memory, `MODULAR-RAG-MCP-SERVER` is the clearest reference.

If the goal is rich substrate-level retrieval context assembled under token budgets, `LightRAG` is the clearest reference.

## Open Questions

- How far can `rag-skill` rely on procedural discipline before it needs a stronger explicit memory layer?
- How long can `deep-rag` conversations stay healthy before replay and raw observation growth become the dominant failure mode?
- Does `MODULAR-RAG-MCP-SERVER` need a richer session memory layer, or is request-scoped context the right product boundary?
- How much operational complexity is justified by `LightRAG`'s richer substrate-level context assembly?
