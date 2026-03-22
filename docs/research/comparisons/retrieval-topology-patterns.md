# Retrieval Topology Patterns

## Comparison Goal

Compare how different local-knowledge RAG systems choose, fetch, and package information before answer generation.

## Compared Projects

- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- retrieval substrate
- preprocessing burden
- runtime narrowing contract
- dominant retrieval unit
- post-retrieval shaping
- runtime state coupling
- serving surface
- dominant failure mode

## Comparison Table

| Dimension | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Main style | knowledge-map-guided file navigation runtime | modular hybrid retrieval service | graph-plus-vector retrieval platform |
| Retrieval substrate | generated file summary plus direct file-system reads | chunked corpus in BM25 plus vector store with dense+sparse fusion | graph storage plus entity/relation/chunk vector stores |
| Preprocessing burden | generate `summary.txt` and mirrored markdown chunks | run full ingestion pipeline to chunk, encode, and store corpus | ingest docs, extract entities/relations, maintain multi-store graph and vector indexes |
| Runtime narrowing contract | model chooses files or directories from the knowledge map, then calls `retrieve_files` | query processor parses keywords and filters, retrievers run dense and sparse search, fusion and optional rerank narrow the result set | query mode picks KG, chunk, or mixed retrieval path; keywords and mode decide which branches run |
| Dominant retrieval unit | full file or full directory payload | ranked chunks with citations | entities, relationships, and chunks; `mix` adds vector chunks on top of KG results |
| Post-retrieval shaping | little runtime compression after retrieval; raw observations go back into the loop | response builder formats ranked chunk results; trace layer records intermediate stages | token-budgeted chunk packing, structured reference lists, and mode-specific context formatting |
| Where state most affects retrieval | frontend-replayed messages plus file-summary map | repo-local config, BM25 indexes, vector-store collections, and trace files | workspace-scoped shared storage, document status, graph stores, vector stores, and cache stores |
| Serving surface | FastAPI backend plus React frontend with function mode and ReAct fallback | stdio MCP server plus placeholder root launcher | core Python library plus FastAPI server, WebUI, and Ollama-style API |
| Strongest fit | corpus navigation when file structure itself is useful and the corpus is modest | service-style retrieval where chunk ranking and MCP exposure matter | persistent knowledge substrate where graph structure and mixed retrieval are first-class |
| Dominant failure mode | retrieval payload blow-up and frontend/backend protocol coupling | bootstrap fragility: launcher drift, dependency drift, and abstraction leaks | operational complexity across backends, storages, and serving modes |

## Retrieval-Control Split

| Question | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Who decides what to retrieve? | mostly the model, using the knowledge map and one retrieval tool | mostly the retrieval stack, after query preprocessing and fusion | mixed control: query mode and keyword extraction decide which retrieval branches activate |
| Is retrieval path explicit to the user? | yes, because tool calls expose file and directory paths | yes at the tool level, but lower-level fusion stays internal | partly; API mode is explicit, but the KG/vector branch mix is more internal |
| Does the retrieval stack depend on a heavy prebuilt substrate? | low to medium | medium | high |
| What matters most for quality? | quality of the knowledge map and path choice | quality of ingestion, chunking, sparse/dense fusion, and rerank | quality of extraction, graph maintenance, vector retrieval, and mode choice |

## Shared Patterns

- all three systems use local durable artifacts as part of retrieval, not just transient in-memory ranking
- all three separate ingestion or preprocessing from query-time answer generation
- all three expose a retrieval contract that is more legible than a pure "embed everything and hope top-k works" baseline

## Key Differences

- `deep-rag` externalizes retrieval choice to the model through path navigation, while `MODULAR-RAG-MCP-SERVER` and `LightRAG` encode much more of retrieval selection in code and storage structure
- `MODULAR-RAG-MCP-SERVER` is the cleanest chunk-ranking service shape of the three, but it currently pays for that with packaging and bootstrap fragility
- `LightRAG` has the richest retrieval substrate because it maintains graph and vector views at once, but it also carries the heaviest operational surface
- `deep-rag` is the least infrastructure-heavy, but it has the weakest built-in protection against oversized retrieval payloads once a broad path is selected

## Conclusions

These projects now suggest three distinct local-knowledge RAG patterns:

- knowledge-map navigation runtime
- modular hybrid chunk-retrieval service
- graph-plus-vector substrate platform

That split is useful because the systems fail for different reasons.

- In knowledge-map navigation runtimes, the main risk is context blow-up after the model chooses a broad retrieval scope.
- In modular hybrid retrieval services, the main risk is bootstrap and packaging drift around an otherwise solid retrieval core.
- In graph-plus-vector platforms, the main risk is operational complexity from maintaining many stores, modes, and backend combinations.

If the corpus is relatively small and file structure is itself informative, `deep-rag` is the most direct pattern.

If the goal is a clean service boundary for ranked retrieval and tool exposure, `MODULAR-RAG-MCP-SERVER` is the strongest architectural reference.

If the goal is a persistent retrieval substrate with richer query modes and graph-aware context assembly, `LightRAG` is the most complete pattern.

## Open Questions

- Can `deep-rag` add post-retrieval compression without losing its transparent file-navigation contract?
- Can `MODULAR-RAG-MCP-SERVER` reconcile its launcher and dependency story enough to make the clean modular core the default user experience?
- Does `LightRAG`'s heavier graph-plus-vector substrate produce meaningfully better answers on real corpora than the simpler alternatives, enough to justify the operational cost?
