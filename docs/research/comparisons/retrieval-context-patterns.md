# Retrieval Context Patterns

## Comparison Goal

Compare how local-knowledge RAG systems build, compress, and hand off retrieval context before or during answer generation.

## Compared Projects

- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- global orientation context
- retrieval-time working context
- compression boundary
- what gets re-injected
- durable context artifacts
- dominant context risk

## Comparison Table

| Dimension | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Global orientation context | `summary.txt` knowledge map injected up front | query pipeline uses collection indexes and settings, not a large global prompt summary | multi-store substrate already holds graph, chunk, relation, and cache context before query time |
| Retrieval-time working context | raw file or directory content returned by `retrieve_files` | ranked retrieval results with metadata and citations | structured entities, relations, and chunks, optionally mixed with vector chunks |
| Main compression boundary | mostly before retrieval via summary generation; little compression after retrieval | compression happens in retrieval ranking and response building | dynamic token budgeting, chunk truncation, and structured context templates before final answer generation |
| What gets re-injected into the next step | full tool observations or ReAct observation messages | usually one-shot retrieval result, not a long conversational loop | structured context string plus reference list for the final answer stage; conversation history is not retrieval memory |
| Durable context artifacts | summary files, mirrored markdown chunks, filesystem corpus, frontend message history | repo-local BM25 indexes, vector-store collections, config, traces | graph stores, vector stores, KV stores, doc-status store, cache store, shared workspace state |
| Dominant context risk | raw observations can become very large and directly inflate the next call | context quality depends on bootstrap health of indexes and stores | context assembly is disciplined, but the overall state substrate is complex and heavy |

## Shared Patterns

- all three systems separate durable retrieval artifacts from the final answer call
- all three avoid relying only on raw conversation history as retrieval memory
- all three make context quality depend heavily on preprocessing or maintained retrieval artifacts

## Key Differences

- `deep-rag` treats context as something the model actively pulls from files during the loop
- `MODULAR-RAG-MCP-SERVER` treats context as something the retrieval service ranks and formats, with little conversational carryover
- `LightRAG` treats context as something assembled from a persistent substrate under explicit token budgets and query modes

## Conclusions

These three projects suggest three retrieval-context patterns:

- active file-pull context
- service-ranked chunk context
- substrate-assembled structured context

That split matters because context failure shows up differently.

- In active file-pull context systems, the main risk is overshooting and reinjecting too much raw text.
- In service-ranked chunk context systems, the main risk is not usually prompt overflow but unhealthy retrieval artifacts or bootstrap drift.
- In substrate-assembled structured context systems, the main risk is complexity in the context-building substrate rather than any single prompt overflow point.

If the goal is to keep retrieval explainable in path-level terms, `deep-rag` is the clearest example.

If the goal is to hand clients compact ranked retrieval outputs, `MODULAR-RAG-MCP-SERVER` is the clearest example.

If the goal is to expose richer context objects under token-budget control, `LightRAG` is the strongest example.

## Open Questions

- Can `deep-rag` add a compression stage without losing the simplicity of direct file retrieval?
- How far can `MODULAR-RAG-MCP-SERVER` scale before chunk-level response formatting needs a stronger secondary summarization layer?
- Does `LightRAG`'s structured context assembly provide meaningfully better downstream answers on real corpora than simpler chunk-ranking systems?
