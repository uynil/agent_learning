# Autonomous Continuation vs Planning

## Comparison Goal

Compare whether local-knowledge systems can keep working within a task on their own, and whether they also perform explicit planning or decomposition rather than only bounded continuation.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- continuation shape
- planning shape
- decomposition owner
- boundedness
- strongest autonomous behavior
- main planning gap

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Continuation shape | procedural continuation through index-following and up to 5 retrieval rounds | model-directed continuation through bounded tool or ReAct loops | service execution continues within one retrieval call, but higher-level continuation stays outside the server | retrieval continues through mode dispatch and substrate branches, not through a visible chat-loop worker |
| Planning shape | almost none; the skill governs workflow but not real decomposition | limited; it can decide the next retrieval step, but not build a rich plan or subquestion tree | minimal; the client must decide the broader sequence of work | low at query time; runtime chooses retrieval branches, but not a user-visible research plan |
| Decomposition owner | the human or invoking agent | the model chooses next retrieval steps, but not a durable explicit plan | the caller or outer orchestrator | the runtime policy and caller-configured mode, not a planner |
| Boundedness | explicit 5-round cap | explicit iteration cap in both function and ReAct modes | naturally bounded by one request | bounded by query mode and pipeline path rather than by an open loop |
| Strongest autonomous behavior | continuing a disciplined local-file workflow without re-asking for each micro-step | retrieving again inside one request after seeing tool output | completing one ranked retrieval service call cleanly | preserving retrieval effectiveness through substrate branches like `mix` even when one branch weakens |
| Main planning gap | no explicit planning, decomposition, or stateful task tree | no rich planner, question decomposition, or long-horizon task structure | no built-in higher-level multi-step orchestration | no visible research planner; autonomy lives in retrieval substrate, not task decomposition |

## Shared Patterns

- none of the four is a true planning-heavy research agent
- all four are stronger at bounded continuation than at explicit decomposition
- all four show that "agentic" local retrieval can exist without a planner, but the flavor of autonomy changes a lot

## Key Differences

- `rag-skill` is autonomy by workflow obedience:
  it can continue, but only along a documented path and without planning its own strategy
- `deep-rag` is the clearest example of bounded autonomous continuation:
  it can decide to retrieve again after seeing tool output, but it still lacks explicit planning machinery
- `MODULAR-RAG-MCP-SERVER` is intentionally weak on both continuation and planning beyond one request:
  it is a retrieval service, not a planner
- `LightRAG` is strongest where substrate-level retrieval continuity matters, but that should not be confused with planning

## Conclusions

These systems suggest four different relationships between continuation and planning:

- workflow continuation without planning
- bounded model continuation without rich planning
- service execution without planner ownership
- substrate continuity without user-visible planning

That distinction matters because people often overread bounded retrieval loops as "planning."

If the goal is to study lightweight continuation without building a planner, `rag-skill` and `deep-rag` are the most relevant references, but for different reasons:
`rag-skill` for procedural obedience and `deep-rag` for in-request model continuation.

If the goal is retrieval infrastructure rather than planning, `MODULAR-RAG-MCP-SERVER` and `LightRAG` are better references.

## Open Questions

- Could `rag-skill` gain a tiny planning layer without losing its transparent workflow character?
- Could `deep-rag` add query decomposition or retrieval budgeting without becoming too heavy?
- Should `MODULAR-RAG-MCP-SERVER` stay planner-free by design?
- Would `LightRAG` benefit from an optional planner above its retrieval modes, or would that belong in an outer orchestrator?
