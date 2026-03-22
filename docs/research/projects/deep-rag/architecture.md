---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: reconcile README and frontend claims with the actual markdown-oriented preprocessing pipeline and backend-driven provider/model contract
---

# Architecture

## Architecture Summary

`deep-rag` is split into four main layers:

- preprocessing for knowledge-base summaries and chunk generation
- a FastAPI backend that mediates prompts, tool loops, and provider calls
- a file-system-backed knowledge base manager
- a React frontend that holds conversation history and visualizes tool calls

The most important architectural decision is to separate "global knowledge understanding" from "content retrieval". A preprocessing script builds the global map, and runtime retrieval only fetches files after the model decides what it needs.

## Module Breakdown

- `backend/main.py`
  Owns HTTP endpoints, chat orchestration entry, provider selection, and SSE streaming.
- `backend/prompts.py`
  Owns system prompt creation, tool schema creation, function-call processing, and ReAct parsing helpers.
- `backend/react_handler.py`
  Runs the prompt-driven ReAct loop for models without native function calling.
- `backend/knowledge_base.py`
  Provides file-summary loading, directory scanning, and file/directory retrieval.
- `backend/llm_provider.py`
  Wraps OpenAI-compatible chat completions behind one streaming interface.
- `backend/config.py`
  Loads provider config, paths, tool mode, and model settings from `.env`.
- `Knowledge-Base-File-Summary/generate.py`
  Precomputes the knowledge map, mirrors small markdown files into the retrieval corpus, and writes chunk files for large markdown documents.
- `frontend/src/App.tsx`
  Holds chat state, streams responses, and replays prior messages into each request.
- `frontend/src/components/ConfigPanel.tsx`
  Edits `.env` through backend config endpoints, but bypasses the shared frontend API abstraction.
- `frontend/src/components/ChatMessage.tsx`
  Visualizes tool calls and can expand or copy large tool-result payloads.

## Execution Flow

1. The frontend sends the full chat history to `/chat`.
2. The backend loads the current file summary from disk.
3. The backend chooses function-calling or ReAct mode based on `settings.tool_calling_mode`.
4. The model receives a system prompt that includes the knowledge map and retrieval rules.
5. If the model calls `retrieve_files`, the backend reads matching files/directories from disk.
6. Tool observations are appended back into the conversation.
7. The loop continues until the model stops calling tools and produces a final answer.

## State and Data Flow

- Global configuration flows from `.env` into `Settings`.
- Knowledge-base metadata flows from `summary.txt` into the system prompt.
- Preprocessed markdown files or chunk files flow into `knowledge_base_chunks`, which is the intended retrieval corpus for runtime reads.
- Retrieved file content flows from `KnowledgeBase.retrieve_files()` into tool results.
- Conversation state flows from the frontend message array into every backend request.
- In function-calling mode, tool calls and tool results are appended to `conversation_messages`.
- In ReAct mode, the raw assistant thought/action text and a synthetic observation user message are appended to `conversation_messages`.
- In the frontend, raw ReAct content is buffered until `<|Final Answer|>` appears, which means the UI depends on backend marker conventions to display clean answers.

## Extension Points

- Provider extension is dynamic: any `{PROVIDER}_MODEL`, `{PROVIDER}_BASE_URL`, and `{PROVIDER}_API_KEY` tuple can become a provider.
- Tool-calling mode can switch between native function calling and prompt-only ReAct.
- Knowledge-base summaries can be regenerated or chunking behavior changed without editing the main backend loop.
- Additional tools could be added by extending the tool schema and tool processing path in `prompts.py`.
- Frontend deployment can theoretically target another backend origin through `VITE_API_BASE_URL`, but the config editor currently does not follow that abstraction.

## Tradeoffs

- The architecture is simple and transparent because everything routes through one retrieval tool.
- The design avoids vector databases and embedding pipelines, which lowers system complexity.
- In exchange, retrieval quality depends heavily on the file-summary map and the model's ability to choose paths correctly.
- Full-file or full-directory retrieval preserves structure, but can produce very large observations without ranking or compression.
- Frontend-driven conversation replay keeps the backend stateless, but limits first-class resumability and server-side memory control.
- The preprocessing layer is conceptually optional in the README, but architecturally close to mandatory because it is responsible for both the knowledge map and the mirrored retrieval corpus.
- The frontend stays visually clean in ReAct mode only because client code understands and strips protocol markers, which increases coupling between transport format and UI logic.
- The provider/model story is only partially wired: values are fetched and carried through state, but the visible UI does not let the user choose them and backend support for per-request model override is incomplete.

## Open Questions

- Should retrieval be followed by an explicit compression step before large observations return to the model?
- Would a second tool for summary-level retrieval or ranking improve scalability?
- Is the current frontend-selected `model` field intentionally unused, or is it an incomplete feature?
- Should config keys for knowledge-base paths be aligned with the actual settings field names?
- Should the config panel stop hardcoding `localhost:8000` so the frontend can actually be deployed behind a different origin or proxy path?
- Should the architecture explicitly treat preprocessing outputs as first-class runtime artifacts instead of documenting chunked documents as merely optional?
