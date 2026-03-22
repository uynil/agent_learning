---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: finish the static context audit around mirrored markdown retrieval, frontend replay, and protocol-specific answer cleanup before deciding whether any more runtime evidence is needed
---

# Context Management

## Context Sources

- current conversation history from the frontend
- system-level file-summary map from `summary.txt`
- current time injected into the system prompt
- retrieved file or directory contents
- ReAct observations or function-call tool results
- optional chunked knowledge-base files generated during preprocessing

## Memory and State Layers

- There is no obvious server-side conversation state store.
- The frontend message array is the main multi-turn memory mechanism.
- `summary.txt` acts as persistent global knowledge orientation.
- The file system itself is the durable memory layer.
- Retrieved observations become transient working memory inside the active chat loop.
- `Knowledge-Base-Chunks/` is a preprocessing memory artifact for large files.
- The intended runtime retrieval corpus is the preprocessed markdown mirror under `knowledge_base_chunks`.

## Retrieval and Selection Strategy

- No vector search or embedding ranking is present in the main path.
- The model selects file paths or directories directly from the knowledge map.
- Retrieval is path-based and exact once the model chooses paths.
- The summary generator precomputes short summaries so the model can decide what to open.
- Bulk retrieval paths are markdown-oriented because directory traversal uses `rglob("*.md")`.

## Context Packing Strategy

- The entire file summary is injected up front into the system prompt.
- Retrieved file contents are appended verbatim to the conversation as tool results or observations.
- There is no explicit compression stage after runtime retrieval.
- Large-file compression happens earlier, during summary/chunk generation, not in the chat loop itself.
- In the current sample runtime, the summary payload is about 2711 characters and the generated function-mode system prompt is about 3657 characters before user messages are added.
- In a live single-file chat trace, both function mode and ReAct mode carried a 22961-character retrieval payload forward without any runtime summarization step.

## Context Update Rules

- The frontend resends the updated message history each turn.
- In function-calling mode, each tool result is appended as a `tool` message.
- In ReAct mode, each observation is appended as a synthetic `user` message.
- No richer memory graph, claim store, or session object is updated on the backend.
- ReAct live traces confirmed that the synthetic observation message includes the raw retrieved file content plus a `Continue with your reasoning.` suffix before the next provider call.
- In the frontend, ReAct final-answer rendering also depends on seeing `<|Final Answer|>` in the buffered stream before the assistant text is surfaced.

## Failure Modes

- The system prompt may become large if the file summary grows with the knowledge base.
- Retrieved directories can produce oversized observations with no follow-up compression.
- Client-side replay means long conversations may grow without backend-side memory discipline.
- If summaries are too compressed, the model may choose the wrong files.
- If summaries are too verbose, the system prompt may become bloated.
- In the live backend run, one directory retrieval already produced about 86.5 KB of text and retrieving `/` produced about 449 KB across 25 files, which makes the no-compression risk concrete rather than theoretical.
- In ReAct mode, the raw observation is also repackaged as a synthetic user turn, so large retrievals directly inflate the next provider call rather than being mediated by a narrower tool-result schema.

## Reusable Patterns

- use a compact knowledge map as pre-retrieval context
- keep durable memory in the file system and summary artifacts
- let the model actively choose retrieval scope
- separate global orientation context from retrieved working context

## Open Questions

- At what corpus size does the file-summary map stop fitting comfortably in the system prompt?
- Does path-based retrieval outperform chunk-first retrieval on realistic noisy corpora?
- Should runtime retrieval results be summarized before being re-injected into the loop?
- Is frontend-driven multi-turn memory enough for longer sessions?
