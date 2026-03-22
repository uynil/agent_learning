# Human Steering vs Autonomous Continuity

## Comparison Goal

Compare how local-knowledge systems divide responsibility between explicit human or operator steering and autonomous continuation inside the system.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- primary steering locus
- continuity carrier
- what the human must still do
- what the system continues on its own
- main failure when autonomy is weak
- main failure when autonomy is strong

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Primary steering locus | the operator or invoking agent following the skill docs | the model inside the bounded retrieval loop | the client or caller deciding when to issue the next request | the retrieval mode, runtime policy, and persistent substrate |
| Continuity carrier | skill protocol, index docs, and local helper artifacts | frontend replay, tool observations, and file-backed retrieval artifacts | persisted collections, traces, and request-scoped tool contracts | workspaces, shared stores, graph/vector state, and query modes |
| What the human must still do | choose the right environment, obey pre-read gates, and recover when tools drift | frame the task and rely on the app shell to keep replaying context | orchestrate multi-step work outside the server if one query is not enough | choose configuration, modes, and operational substrate wisely |
| What the system continues on its own | very little beyond whatever the invoking agent remembers and reuses | retrieval can continue inside one request without asking the user again | retrieval execution continues cleanly within one call, but not usually across calls | retrieval state and corpus-level continuity survive across many operations |
| Main failure when autonomy is weak | workflow success becomes too dependent on operator discipline | long sessions depend too heavily on client replay and manual app assumptions | multi-step analysis has to be rebuilt outside the service boundary | heavy substrate may still require too much operator knowledge to tune well |
| Main failure when autonomy is strong | not the main risk here; the bigger risk is under-automation | the model can choose broad retrieval paths that bloat context | not the main risk here; service remains intentionally conservative | autonomy moves into many stores and modes, which increases hidden operational complexity |

## Shared Patterns

- none of the four is fully autonomous in the "research agent" sense
- all four still depend on human choices somewhere:
  skill use, app framing, client orchestration, or infrastructure configuration
- all four show that continuity can come from artifacts and substrate, not only from a long-running agent session

## Key Differences

- `rag-skill` sits closest to the human-steered end:
  continuity exists, but it is mostly carried by procedure and operator discipline
- `deep-rag` moves further toward autonomous continuation at query time:
  once a request begins, the model can keep retrieving inside the same bounded loop
- `MODULAR-RAG-MCP-SERVER` is conservative:
  it automates retrieval execution well, but expects higher-level orchestration to live outside the server
- `LightRAG` automates continuity at the substrate level more than at the visible agent-loop level:
  the system preserves retrieval readiness and state, but not chiefly through a chat-like autonomous worker

## Conclusions

These projects suggest four continuity styles:

- human-steered procedural continuity
- model-steered bounded continuity
- caller-steered service continuity
- substrate-steered persistent continuity

That split matters because people often say a system is "agentic" when they really mean one of several different things:
it can keep working inside one request, it can remember across requests, or it reduces operator burden at the infrastructure layer.

If the goal is transparent human control with lightweight protocol help, `rag-skill` is the clearest reference.

If the goal is bounded autonomy inside one answer flow, `deep-rag` is the clearest reference.

If the goal is explicit orchestration boundaries and conservative service behavior, `MODULAR-RAG-MCP-SERVER` is the clearest reference.

If the goal is durable continuity through retrieval substrate rather than through a visible agent loop, `LightRAG` is the clearest reference.

## Open Questions

- At what point would `rag-skill` gain more from explicit session artifacts than it would lose in simplicity?
- Can `deep-rag` gain better bounded autonomy without paying so much in raw-context growth?
- Should `MODULAR-RAG-MCP-SERVER` remain caller-steered, or would an optional higher-level continuation layer add real value?
- How often does `LightRAG`'s substrate continuity translate into better user outcomes rather than just more infrastructure?
