---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: compare this loop structurally against stronger deep-research projects already studied here, instead of adding more runtime probes
---

# Deep Research Mode

## Research Loop Overview

`deep-rag` supports iterative retrieval, but its loop is narrower than the "deep research" systems already studied in this repository. It does not build a report plan, decompose broad questions into subquestions, or maintain a long-lived research state. Instead, it repeats:

1. inspect the file-summary map
2. decide whether retrieval is needed
3. retrieve files or directories
4. continue reasoning with the new observation
5. answer when enough evidence is present

## Question Rewrite Strategy

- No dedicated query rewrite stage is visible in the code.
- The model may implicitly reformulate what it needs through repeated retrieval choices.
- In ReAct mode, that reformulation is hidden inside the `<|Thought|>` step rather than surfaced as a separate artifact.

## Decomposition Strategy

- Decomposition is retrieval-scope-based, not subquestion-based.
- The model can narrow from the whole summary to a directory, then to a file.
- This is a navigation strategy more than a research-plan strategy.

## Evidence Accumulation

- Evidence accumulates through appended tool observations.
- There is no dedicated evidence table, claim graph, or round summary artifact.
- Final answers are grounded in the current conversation plus retrieved file contents.

## Reflection and Revision

- ReAct mode provides a reasoning/action loop, but not a formal critique stage.
- Function-calling mode provides iterative retrieval without explicit reflection text.
- There is no specialized reviewer or reviser step.

## Stop Conditions

- Function-calling mode stops when the model no longer calls tools or reaches `max_iterations = 10`.
- ReAct mode stops when no action is detected or `max_iterations = 5` is reached.
- There is no adaptive stop logic based on evidence sufficiency beyond the model's own decision.

## Resumability and Context Handoff

- The backend is effectively stateless across requests.
- Continuation depends on the client resending conversation history.
- No explicit deep-research round summary or resume artifact exists.

## Reusable Patterns

- use a knowledge map to support iterative retrieval
- let the model refine retrieval scope round by round
- support weaker models with a prompt-only ReAct fallback

## Static Classification

- Based on the code structure alone, this looks closer to active file-navigation RAG than to a full deep-research agent.
- Its loop is retrieval-iterative, but not planner-driven.
- The strongest "research-like" behavior is evidence accumulation through repeated retrieval, not explicit question decomposition or synthesis planning.

## Open Questions

- Is iterative file navigation enough to classify this as deep research, or is it better described as active RAG?
- Would explicit question decomposition materially improve the current design?
- Does ReAct mode add meaningful research depth, or mostly just a compatibility protocol for models without native tool calling?
