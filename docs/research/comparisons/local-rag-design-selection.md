# Local RAG Design Selection

## Goal

Turn the current comparison work into a practical selection guide for local-knowledge RAG system design.

## Compared Reference Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## One-Line Summary

- Pick `rag-skill` when you want a lightweight, documentation-governed retrieval workflow over a local mixed-format corpus.
- Pick `deep-rag` when you want the model to actively navigate a file-backed corpus at query time.
- Pick `MODULAR-RAG-MCP-SERVER` when you want a cleaner retrieval service boundary with ranked chunk results and MCP exposure.
- Pick `LightRAG` when you want a persistent retrieval substrate with graph-aware and multi-mode retrieval behavior.

## Decision Table

| If You Need | Best Reference | Why |
| --- | --- | --- |
| Lightweight local retrieval with minimal infrastructure | `rag-skill` | workflow discipline, manual indexes, and file-type guidance do most of the work |
| Visible multi-step retrieval inside one answer flow | `deep-rag` | the model can decide to retrieve again during the same request |
| Stable machine-to-machine retrieval service | `MODULAR-RAG-MCP-SERVER` | retrieval is exposed as a service/tool primitive rather than a visible agent loop |
| Rich retrieval modes over a long-lived knowledge substrate | `LightRAG` | graph, vector, and mode-based retrieval are first-class parts of the system |
| Minimal infrastructure and high retrieval transparency | `rag-skill` | the control plane is visible in skill docs and local file operations |
| App-shaped interactive retrieval with transparent file navigation | `deep-rag` | file paths and directories are explicit, and the retrieval story is easy to inspect |
| Strong chunk ranking and citation-oriented output | `MODULAR-RAG-MCP-SERVER` | dense+sparse fusion plus response formatting are central to the design |
| Heavier retrieval substrate that can survive sparse KG extraction | `LightRAG` | `mix` can still carry useful retrieval through the chunk/vector branch |

## Steering and Continuity Table

| If You Prefer | Best Reference | Why |
| --- | --- | --- |
| human-steered workflow continuity | `rag-skill` | continuity is carried by skill protocol, indexes, and reusable local artifacts |
| model-steered bounded continuity | `deep-rag` | the model can keep retrieving within one request while the frontend keeps the session alive |
| caller-steered service continuity | `MODULAR-RAG-MCP-SERVER` | the retrieval service stays conservative and leaves higher-level continuity to the client |
| substrate-steered persistent continuity | `LightRAG` | durable workspaces and multi-store state preserve retrieval readiness across operations |

## Selection Axes

### 1. Where should retrieval intelligence live?

- If it should live mostly in workflow docs and operator discipline, use the `rag-skill` pattern.
- If it should live mostly in the model at query time, use the `deep-rag` pattern.
- If it should live mostly in retrieval code and ranking logic, use the `MODULAR-RAG-MCP-SERVER` pattern.
- If it should live mostly in a maintained graph/vector substrate plus explicit runtime modes, use the `LightRAG` pattern.

### 2. What kind of context can your system afford?

- If you want extremely selective, procedural local reads and can tolerate manual discipline, `rag-skill` is the best fit.
- If you can tolerate occasional large raw file observations in exchange for simpler architecture, `deep-rag` is the best fit.
- If you want compact ranked chunks and a narrower retrieval handoff, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If you want richer structured context under explicit token budgeting, `LightRAG` is the best fit.

### 3. How much serving complexity can you accept?

- If you want no dedicated backend at all and can live inside a skill or agent environment, `rag-skill` is the best fit.
- If you want an app-shaped experience and can manage frontend/backend coupling, `deep-rag` is the best fit.
- If you want a protocol-first surface and can pay some bootstrap cost, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If you want one core that can power API, WebUI, and compatibility surfaces, `LightRAG` is the best fit.

### 4. How agentic should query-time behavior be?

- If you want agenticity to come from documented workflow obedience rather than a runtime loop controller, `rag-skill` is the best fit.
- If you want visible bounded agent loops, `deep-rag` is the best fit.
- If you want retrieval as a reliable primitive, not an agent, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If you want a pipeline-rich system that is not overtly agentic to callers, `LightRAG` is the best fit.

### 5. Where should session continuity live?

- If continuity should live in reusable artifacts and operator habits, `rag-skill` is the best fit.
- If continuity should live in an interactive client replaying a bounded retrieval loop, `deep-rag` is the best fit.
- If continuity should live mostly outside the retrieval service, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If continuity should live in a persistent retrieval substrate and workspace state, `LightRAG` is the best fit.

### 6. Do you need autonomous continuation or actual planning?

- If you only need workflow continuation with clear operator control, `rag-skill` is the best fit.
- If you need bounded autonomous continuation inside one request, `deep-rag` is the best fit.
- If you do not need planner-like behavior inside the retrieval system at all, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If you need substrate continuity more than planner behavior, `LightRAG` is the best fit.

### 7. Should resume depend more on artifacts or on live runtime state?

- If you want the strongest artifact-first resume story, `rag-skill` is the best fit.
- If you can accept a mixed artifact-plus-runtime resume model for better interactivity, `deep-rag` is the best fit.
- If you want re-runnable retrieval over persisted collections with little conversational runtime dependency, `MODULAR-RAG-MCP-SERVER` is the best fit.
- If you want resume to inherit the strength and cost of a persistent retrieval substrate, `LightRAG` is the best fit.

## Anti-Patterns to Avoid

- Do not choose the `rag-skill` pattern if you need reliable automation across unstable local environments, drifting indexes, or large corpora without stronger runtime enforcement.
- Do not choose the `deep-rag` pattern if your corpus is large enough that raw file or directory retrieval will routinely blow up context.
- Do not choose the `MODULAR-RAG-MCP-SERVER` pattern if you are unwilling to own bootstrap, packaging, and dependency discipline.
- Do not choose the `LightRAG` pattern if you do not want to own a heavy state substrate with many stores, modes, and backend combinations.

## Practical Heuristics

- Start from `rag-skill` if your corpus is modest, mixed-format, and operated by someone who can benefit from explicit retrieval procedure without standing up a service.
- Start from `deep-rag` if your corpus is modest, file structure matters, and you want retrieval behavior that is easy to inspect end to end.
- Start from `MODULAR-RAG-MCP-SERVER` if your core requirement is to expose retrieval cleanly to other tools or agents.
- Start from `LightRAG` if your core requirement is retrieval quality and mode richness over a maintained knowledge substrate, and you can afford the operational surface.

## Related Comparison Pages

- [retrieval-topology-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/retrieval-topology-patterns.md)
- [serving-surface-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/serving-surface-patterns.md)
- [prompt-control-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/prompt-control-patterns.md)
- [local-knowledge-context-management-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/local-knowledge-context-management-patterns.md)
- [retrieval-context-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/retrieval-context-patterns.md)
- [agent-loop-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/agent-loop-patterns.md)
- [state-management-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/state-management-patterns.md)
- [session-management-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/session-management-patterns.md)
- [human-steering-vs-autonomous-continuity.md](/Users/uynil/projects/agents/docs/research/comparisons/human-steering-vs-autonomous-continuity.md)
- [autonomous-continuation-vs-planning.md](/Users/uynil/projects/agents/docs/research/comparisons/autonomous-continuation-vs-planning.md)
- [artifact-vs-runtime-resume-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/artifact-vs-runtime-resume-patterns.md)
- [local-knowledge-reference-cards.md](/Users/uynil/projects/agents/docs/research/comparisons/local-knowledge-reference-cards.md)

## Open Questions

- At what corpus size or workflow complexity does `rag-skill` stop being the most economical pattern?
- At what corpus size does `deep-rag` stop being the most economical pattern?
- Can `MODULAR-RAG-MCP-SERVER` preserve its clean service shape while becoming easier to bootstrap by default?
- Does `LightRAG`'s heavier substrate consistently outperform the lighter patterns enough to justify its operational cost?
