# Agent Loop Patterns

## Comparison Goal

Compare how local-knowledge RAG systems iterate, stop, and hand control between retrieval and answer stages.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- loop shape
- who drives iteration
- stop condition
- retry or fallback style
- degree of agenticity
- best-fit task shape

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Core loop shape | manual or agent-followed retrieval workflow with up to five narrowing rounds over indexes and file handlers | bounded tool loop in function mode or ReAct mode | mostly one-shot retrieval service call per tool invocation | staged pipeline, but not a user-visible multi-step agent loop for standard queries |
| Who drives iteration | the skill instructions and the agent's obedience to them | the model, within `max_iterations` bounds | retrieval code; user or client issues another call if needed | code-defined pipeline and mode selection; internal LLM substeps exist, but not a visible autonomous retrieval loop |
| Stop condition | answer found, branch exhausted, or 5-round cap reached | no more tool call or action detected, or iteration cap reached | return formatted retrieval result or empty response | retrieval context assembled and answer generated, or no-result/failure path hit |
| Retry or fallback style | refine keywords, switch branch, or swap file-type workflow; tool availability can force ad hoc fallbacks | switch between native function mode and ReAct fallback; provider errors stay inside stream contract | dense/sparse graceful degradation and tool-level error responses over MCP | mode-based branching, keyword fallback to raw query, and chunk-only success path even when KG extraction is sparse |
| Degree of agenticity | medium in behavior, but mostly prompt-governed rather than runtime-orchestrated | highest of the four; retrieval can be multi-step and model-directed inside one request | lowest of the four; closer to a retrieval primitive exposed as a tool | medium internally, low externally; several LLM-assisted transformations exist but the user-facing query path feels like structured retrieval, not an autonomous agent |
| Best fit | local knowledge tasks where workflow discipline matters more than infrastructure | queries that benefit from iterative file navigation | clients that want a stable machine-to-machine retrieval call | users who want configurable retrieval modes over a persistent knowledge substrate |

## Shared Patterns

- none of the four is a free-form open-ended agent loop
- all four use bounded, repeatable control structures rather than indefinite conversational wandering
- all four couple retrieval and answer generation differently, but the ownership of the loop moves between docs, model, service, and pipeline

## Key Differences

- `rag-skill` is a protocol loop:
  the loop exists because the skill tells the agent how to iterate, not because the runtime owns a true loop controller
- `deep-rag` is the only one where the model visibly decides whether to retrieve again inside the same user request
- `MODULAR-RAG-MCP-SERVER` is closer to a retrieval action than an agent loop; its "looping" mostly lives in the client or caller
- `LightRAG` has the richest internal pipeline but the least agent-like external query experience among the four, because the branching is mostly code-defined by mode and pipeline stage

## Conclusions

These projects suggest four loop families:

- documentation-governed protocol loop
- model-directed bounded retrieval loop
- service-call retrieval primitive
- pipeline-directed retrieval flow

That split matters because "agentic" means very different things here.

- `rag-skill` is agentic mostly by instruction protocol, not by runtime control plane.
- `deep-rag` is agentic at query time.
- `MODULAR-RAG-MCP-SERVER` is not really agentic at query time; it is a service-oriented retrieval core.
- `LightRAG` is pipeline-rich but not especially agentic from the caller's perspective.

If the goal is to study an agent loop enforced by skill instructions over local files, `rag-skill` is the most relevant reference.

If the goal is visible multi-step retrieval inside one answer flow, `deep-rag` is the most relevant reference.

If the goal is a stable retrieval building block that other systems can call, `MODULAR-RAG-MCP-SERVER` is the most relevant reference.

If the goal is configurable retrieval behavior without exposing an explicit agent loop to the user, `LightRAG` is the most relevant reference.

## Open Questions

- How stable is `rag-skill`'s protocol loop when the agent partially ignores the documented pre-read gates?
- How much does `deep-rag` gain from model-directed iteration versus a more deterministic retrieval policy?
- Does `MODULAR-RAG-MCP-SERVER` need any higher-level loop abstraction, or is its current service shape the right boundary?
- At what point does `LightRAG` become "agentic enough" internally that it should be compared to agent loops rather than only to retrieval pipelines?
