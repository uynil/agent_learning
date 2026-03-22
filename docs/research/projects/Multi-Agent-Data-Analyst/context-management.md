---
project: Multi-Agent-Data-Analyst
source_path: examples/Multi-Agent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: run a live query to see whether the compacted context is sufficient for non-trivial data analysis requests
---

# Context Management

## What Context Exists

- The user prompt itself.
- `OrchestratorDependency.user_prompt`, `model`, `data`, `db_connection`, and `usage_limits`.
- Database schema text from SQLAlchemy introspection.
- A compact `DataFrame` summary with shape, columns, dtypes, missingness, and basic statistics.
- A small sample of query results before chart generation.
- Gradio chat history for display continuity in the UI.
- Agent-local state in `SQLDataAnalysisAgent._state`.
- Agent-local response fields in `DataVisualizationAgent`.

## What Is Not Present

- No persistent conversation memory across requests.
- No retrieval layer or vector store in the observed main path.
- No explicit context packing strategy beyond manual summaries and samples.
- No resumable session object that can be rehydrated later.
- No evidence of a long-context management policy or truncation policy beyond selecting the top rows / summary text.

## Context Compression Strategy

The project compresses data context manually instead of storing broad history.

- SQL requests receive the database schema, not full table dumps.
- Visualization requests receive a short head sample and column types, not the whole DataFrame.
- The UI preserves the visible chat thread, but that is a display artifact rather than a memory store.

This is a practical pattern for small or medium tabular tasks, but it can become thin for wide schemas or large data files.

## State Boundaries

- `SQLDataAnalysisAgent` keeps request-local state in a dictionary, then resets it for each invocation.
- `DataVisualizationAgent` resets its own response, code, and figure on every call.
- `process_user_input_stream` reconstructs context for each request and does not accumulate hidden memory.
- The application does not appear to maintain a backend session store.

## Inferences

- The system is using "context compression" more than "memory management".
- Context quality comes from schema summaries, sample rows, and task-specific prompts rather than from durable agent memory.
- The UI history makes the app feel conversational, but the backend remains mostly stateless across turns.

## Open Questions

- Would a persistent per-session summary improve multi-turn analysis quality?
- How often does the current summary strategy omit details that matter for difficult requests?
- Should the project add an explicit memory layer if future work needs deeper iterative analysis?

