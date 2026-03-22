---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: finish the static protocol audit between backend SSE output and frontend stream parsing, especially around ReAct marker handling and provider/model controls
---

# LLM Orchestration

## LLM Call Map

| Stage | Component | Responsibility |
| --- | --- | --- |
| 1 | `backend/main.py` | build system prompt and select tool mode |
| 2 | `LLMProvider.chat_completion()` | call the configured provider with streaming enabled |
| 3 | function mode loop | accumulate tool call deltas, execute `retrieve_files`, append tool results, continue |
| 4 | ReAct loop | stream free-form reasoning, parse action tags, execute `retrieve_files`, inject observation, continue |
| 5 | final answer stage | stop when the model no longer calls tools and return the answer |

## Scheduling Pattern

- The runtime is single-session and iterative.
- Function-calling mode is a bounded tool loop with `max_iterations = 10`.
- ReAct mode is a bounded thought/action loop with `max_iterations = 5`.
- There is no parallel tool execution in the main path.
- The only tool in the current design is file retrieval.

## Structured Output Strategy

- In function-calling mode, the model is expected to emit native tool call deltas compatible with OpenAI-style streaming.
- In ReAct mode, the model is expected to follow `<|Thought|>`, `<|Action|>`, `<|Action Input|>`, and `<|Final Answer|>` conventions.
- Tool arguments are JSON for both modes.
- The final answer is plain natural language, streamed back to the client.

## Context Handoff

- The knowledge-map summary is injected once at the system level.
- The frontend sends the full prior message list into every `/chat` call.
- In function-calling mode, tool observations become `role="tool"` messages appended to `conversation_messages`.
- In ReAct mode, observations are re-injected as a synthetic `role="user"` message telling the model to continue reasoning.
- No separate server-side session object is maintained.
- In the shipped frontend, ReAct content is buffered until `<|Final Answer|>` appears, so the visible answer depends on frontend-side cleanup rather than on a clean backend SSE contract by itself.

## Reliability Mechanisms

- The function loop is bounded by iteration count.
- The ReAct loop is bounded by iteration count.
- Tool calls are surfaced to the frontend for inspection.
- `LLMProvider` gracefully streams connection errors back as content instead of crashing the stream.
- The system prompt explicitly tells the model to answer "I don't know" only after diligent retrieval.
- In the current live run, a provider-side `400 Bad Request` was surfaced to the client as streamed content plus a final `done` event, confirming that the error path stays inside the SSE contract.
- A local OpenAI-compatible stub provider was enough to validate the full happy path in both orchestration modes.
- Function mode live validation showed the intended SSE order: `tool_calls` -> `tool_results` -> final answer content -> `done`.
- ReAct mode live validation showed a different order: reasoning-tag content -> `tool_calls` -> `tool_results` -> final answer content -> `done`.
- Static frontend review shows that raw ReAct reasoning content is intentionally withheld from the chat transcript until the client sees `<|Final Answer|>`, which partially masks backend protocol noise in the default UI.

## Strengths

- The orchestration is easy to trace.
- Supporting both native function calling and ReAct makes the project portable across providers.
- The tool layer is minimal, which reduces integration complexity.
- The system prompt and retrieval tool are tightly aligned.
- The runtime can work against a generic OpenAI-compatible provider contract, which lowers the amount of backend code tied to any one vendor.
- The shipped frontend already contains protocol-aware cleanup logic for ReAct answers, which keeps the visible chat cleaner than the raw SSE stream.

## Limitations

- There is no ranking, compression, or secondary selection step after raw file retrieval.
- ReAct parsing is string-format fragile compared with native tool calling.
- The orchestration is limited to one retrieval tool, so complex workflows are still model-driven rather than code-structured.
- The `model` field supplied by the client is not clearly used in the backend path.
- The shipped `.env` does not support a successful chat run out of the box because all provider API keys remain placeholders.
- The documented `KNOWLEDGE_BASE_PATH` and `KNOWLEDGE_BASE_CHUNKS_PATH` variable names do not match the runtime settings fields actually consumed by the backend.
- ReAct mode still produces a noisy transport protocol, but the default frontend cleans it up with marker-aware parsing; this means correctness depends on a tight coupling between backend prompt format and client parsing rules.
- The frontend carries provider and model values through state, but does not actually render selector controls, so the orchestration surface is narrower than the state model suggests.
- Runtime parameter updates are not uniformly centralized: `backend.main` can rebind its own `settings`, while `LLMProvider` still reads `temperature` and `max_tokens` from `backend.config.settings`, so post-save behavior may diverge by field.

## Questions for Further Research

- How often does ReAct parsing fail on real providers?
- Do large directory retrievals make the loop unstable or expensive?
- Would a second-stage summarization loop improve answer quality without losing the current simplicity?
- Should the backend support per-request model overrides rather than provider-only selection?
- Should the backend or frontend strip ReAct protocol markers before display so the fallback mode feels like a normal answer stream?
- Should the backend emit a cleaner ReAct-specific event shape so the frontend does not need to infer answer boundaries from `<|Final Answer|>` tags?
- Should `.env` hot reload recreate settings-dependent singletons so orchestration parameters do not drift across modules after a config edit?
