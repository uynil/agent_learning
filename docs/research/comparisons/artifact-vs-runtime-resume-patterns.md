# Artifact vs Runtime Resume Patterns

## Comparison Goal

Compare whether local-knowledge systems are resumed mainly through durable artifacts or through live runtime/session state.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- main resume carrier
- artifact dependence
- runtime dependence
- what must stay alive
- what can be recreated later
- main resume risk

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Main resume carrier | corpus files, `data_structure.md`, and extracted helpers | frontend message replay plus `summary.txt`, mirrored corpus, and config artifacts | collections, traces, indexes, and config files | workspaces, shared stores, caches, graph/vector indexes, and pipeline status |
| Artifact dependence | very high | medium to high | very high | high |
| Runtime dependence | very low | medium because interactive continuity depends on the client keeping the chat alive | low for standard retrieval, because each call is mostly independent | medium to high, because substrate health and store coordination matter even though the data is durable |
| What must stay alive | little beyond access to the same working directory and tools | the browser or client session if you want conversational continuity, plus the backend if you want uninterrupted interaction | almost nothing beyond the service being callable against the same persisted data | the storage substrate and workspace semantics need to stay coherent |
| What can be recreated later | nearly the whole workflow can be replayed from the same artifacts | the backend loop can be restarted, but conversational feel weakens if replay history is lost | request state can be recreated by rerunning the query against the same collections | query-time behavior can be recreated if the persistent substrate remains healthy |
| Main resume risk | operator forgets the exact path taken, or environment drift changes tool behavior | chat continuity depends on frontend replay, and config hot-reload drift can blur what "same runtime" means | good artifact recovery but weak interactive continuation | strong persistence, but recovery semantics depend on many storage and coordination layers staying consistent |

## Shared Patterns

- all four depend on artifacts more than on opaque in-memory reasoning state
- none of the four is primarily a "keep one long agent process alive forever" design
- all four can be resumed after interruption, but they resume from different layers

## Key Differences

- `rag-skill` is the most artifact-centric:
  the workflow can be resumed by re-opening the same corpus, indexes, and helper files
- `deep-rag` is the most mixed:
  artifacts keep retrieval meaningful, but interactive continuity is strongly runtime- and client-dependent
- `MODULAR-RAG-MCP-SERVER` is artifact-centric at the service layer:
  traces and collections matter more than any live session
- `LightRAG` is also artifact-heavy, but its effective resume model depends more on live substrate integrity than `rag-skill` or `MODULAR-RAG-MCP-SERVER`

## Conclusions

These systems suggest four resume patterns:

- artifact-first workflow resume
- hybrid artifact-plus-runtime resume
- artifact-first service resume
- substrate-backed runtime-aware resume

That split helps explain why two systems can both be "resumable" while feeling very different to operate.

If the goal is replayable work from transparent local artifacts, `rag-skill` is the clearest reference.

If the goal is conversational continuity over local retrieval, `deep-rag` is the clearest reference, but it also has the most runtime-sensitive resume story.

If the goal is re-runnable retrieval over persisted collections and traces, `MODULAR-RAG-MCP-SERVER` is the clearest reference.

If the goal is durable retrieval capability across many operations, `LightRAG` is the clearest reference, provided the substrate remains healthy.

## Open Questions

- How much environment drift can `rag-skill` tolerate before artifact-first resume becomes unreliable?
- Can `deep-rag` gain a cleaner server-side resume model without losing its lightweight app shape?
- Would `MODULAR-RAG-MCP-SERVER` benefit from a stronger notion of continuation across related calls?
- How much store-coordination complexity is acceptable before `LightRAG`'s resume model becomes too operationally expensive?
