# Session Management Patterns

## Comparison Goal

Compare how local-knowledge systems represent a session boundary, resume prior work, and decide what survives across requests or agent turns.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- session owner
- main session artifact
- cross-turn memory style
- resume boundary
- failure recovery shape
- operational risk

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Session owner | the invoking agent or user workflow | mostly the frontend client plus transient backend loop state | the MCP client or caller; the server is largely request-scoped | the runtime substrate and workspace-scoped stores, not a chat client alone |
| Main session artifact | current explored files, extracted helper files, and whatever the agent carries from the skill flow | browser-side message history, `summary.txt`, mirrored chunk corpus, and mutable `.env` | per-request traces, collection data, config files, and persistent indexes | workspace defaults, `pipeline_status`, graph/vector/KV stores, caches, and query params |
| Cross-turn memory style | manual and procedural; no first-class session object | replay-based; the frontend resends prior messages every turn | weak conversational memory; each call is mostly independent unless the client reuses context | substrate-backed; retrieval state persists even though conversational state is optional and secondary |
| Resume boundary | resume by re-reading indexes and artifacts, not by restoring a runtime session | resume by keeping browser history and file artifacts; backend itself is thin | resume by reissuing calls against the same collections and persisted stores | resume by reconnecting to the same workspace and persisted stores |
| Failure recovery shape | re-open the same corpus branch, extraction output, or reference doc and continue manually | restart the request loop or keep chatting if the frontend still holds history; hot-reload drift can make "same session" ambiguous | rerun the tool call; inspect traces and collection artifacts for diagnosis | rely on store consistency, workspace isolation, and shared-state coordination |
| Operational risk | no explicit session contract, so behavior depends heavily on agent discipline | long-history replay growth and split config state can make session semantics fuzzy | thin session model may be fine for retrieval, but weak for interactive iterative research | strong persistence comes with coordination and storage-consistency complexity |

## Shared Patterns

- none of the four uses a rich server-side conversational session model as its core identity
- all four depend on external artifacts more than on opaque in-memory chat state
- all four make "resume" mean something different from "restore the exact same runtime object"

## Key Differences

- `rag-skill` has the weakest explicit session boundary:
  continuity lives mostly in operator discipline and reusable artifacts
- `deep-rag` has the strongest interactive feel, but its session is really client replay plus file-backed retrieval artifacts rather than a robust backend session object
- `MODULAR-RAG-MCP-SERVER` treats session continuity as mostly the caller's problem; the server preserves traces and collections, not an evolving dialogue
- `LightRAG` has the strongest persistent runtime substrate, but that persistence is centered on workspaces and stores rather than on a human-readable conversational session

## Conclusions

These projects suggest four session-management styles:

- procedural session continuity
- client-replayed interactive session
- request-scoped service session
- substrate-backed workspace session

That distinction matters because "session support" can mean very different things:
remembering a chat, preserving a corpus, reusing retrieval state, or simply leaving enough artifacts behind to continue later.

If the goal is low-infrastructure workflow continuity, `rag-skill` is the clearest reference.

If the goal is interactive chat continuity over local retrieval, `deep-rag` is the clearest reference.

If the goal is machine-to-machine retrieval with traceable independent calls, `MODULAR-RAG-MCP-SERVER` is the clearest reference.

If the goal is durable retrieval state across many operations and workspaces, `LightRAG` is the clearest reference.

## Open Questions

- Should `rag-skill` ever grow a first-class session artifact, or would that undercut its lightweight value?
- Does `deep-rag` need real backend session state, or is frontend replay the right complexity boundary?
- Would `MODULAR-RAG-MCP-SERVER` benefit from an optional caller-managed session token or trace-linked continuation model?
- How much operational overhead is acceptable before `LightRAG`'s substrate-backed continuity stops feeling "light"?
