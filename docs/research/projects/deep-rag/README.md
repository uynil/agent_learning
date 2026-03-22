---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: if needed, turn the four review-grade candidates into formal review findings or a concrete patch plan, instead of continuing broad exploratory analysis
---

# deep-rag

## Project Summary

`deep-rag` is a knowledge-base QA system that tries to replace chunk-first semantic retrieval with a "knowledge map + active navigation" pattern. Instead of retrieving embeddings by similarity, it gives the model a summary of the whole repository and lets the model call a `retrieve_files` tool to pull specific files, directories, or the entire corpus.

## Quick Classification

- Style: tool-using RAG system with optional ReAct mode
- Best fit: file-system-backed knowledge navigation and multi-step retrieval
- Deep research behavior: partial; it supports iterative retrieval loops, but not a rich planner or question-decomposition workflow

## Key Files

- `examples/deep-rag/backend/main.py`
- `examples/deep-rag/backend/prompts.py`
- `examples/deep-rag/backend/react_handler.py`
- `examples/deep-rag/backend/knowledge_base.py`
- `examples/deep-rag/backend/llm_provider.py`
- `examples/deep-rag/backend/config.py`
- `examples/deep-rag/backend/models.py`
- `examples/deep-rag/Knowledge-Base-File-Summary/generate.py`
- `examples/deep-rag/Knowledge-Base-File-Summary/Summary Prompt.md`
- `examples/deep-rag/frontend/src/App.tsx`
- `examples/deep-rag/frontend/src/api.ts`

## Current Status

The first-pass analysis is complete.
The backend boot path, retrieval path, function-mode chat path, and ReAct-mode chat path have all been validated locally, and the remaining important work is now comparative rather than foundational.
The main unresolved risks are oversized retrieval payloads, config/docs drift, and frontend/backend contract fragility, not uncertainty about whether the system basically works.

## Current Research Themes

- architecture and backend/frontend split
- prompt function and file-navigation mechanism
- tool-calling orchestration in function and ReAct modes
- context management through knowledge maps, observations, and client-side message replay
- limits of calling this a deep-research system

## Key Research Questions

- How much of the system's capability comes from the knowledge-map summary versus the iterative tool loop?
- Is multi-turn memory truly a server-side capability, or mainly a frontend message-replay pattern?
- Does the env/config layer behave as documented, especially for knowledge-base path settings?
- How well does the approach scale when retrieval returns very large file or directory payloads?

## Next Action

- Turn the now-stable review-grade candidates into a proper review pass or patch plan.

## Research Notes

- [architecture.md](architecture.md)
- [llm-orchestration.md](llm-orchestration.md)
- [context-management.md](context-management.md)
- [deep-research-mode.md](deep-research-mode.md)
- [prompts-analysis.md](prompts-analysis.md)
- [notes.md](notes.md)

## Observed Facts

- The FastAPI app exposes `/chat`, `/knowledge-base/info`, `/system-prompt`, and config endpoints in `backend/main.py`.
- The backend builds the system prompt from a generated file-summary map via `create_system_prompt()` or `create_react_system_prompt()`.
- The only tool in the main path is `retrieve_files`, defined in `backend/prompts.py`.
- Function-calling mode loops over streaming tool calls up to `max_iterations = 10`.
- ReAct mode loops up to `max_iterations = 5` and parses `<|Action|>` and `<|Action Input|>` markers from model output.
- `KnowledgeBase.retrieve_files()` can return specific files, whole directories, or all files under `/`.
- `KnowledgeBase` reads from `settings.knowledge_base_chunks`, while summary generation reads from `settings.knowledge_base`.
- The summary generator creates `summary.txt` and can split large files into chunk files under `Knowledge-Base-Chunks/`.
- The frontend sends the full message history on every `/chat` request through `frontend/src/App.tsx`.
- `ChatRequest` includes an optional `model`, but `backend/main.py` only uses `request.provider` when constructing `LLMProvider`.
- The README and `.env.example` document `KNOWLEDGE_BASE_PATH` and `KNOWLEDGE_BASE_CHUNKS_PATH`, while `backend/config.py` defines settings fields named `knowledge_base` and `knowledge_base_chunks`.
- The README configuration section still documents `KNOWLEDGE_BASE_PATH`, while the current settings model only binds the shorter field name `knowledge_base`.
- After creating a local backend venv and installing `requirements.txt`, the FastAPI backend booted successfully on a local test port.
- A live call to `/` returned a healthy status and listed providers `anthropic`, `custom`, `google`, and `openai`.
- A live call to `/config` returned `google` as the default provider and `gemini-2.5-flash-lite` as the default model.
- A live call to `/knowledge-base/info` returned a file-summary map of length 2711 characters and a file tree rooted in `Knowledge-Base/`.
- A live call to `/system-prompt` in function mode returned a prompt of length 3657 characters.
- A live call to `/knowledge-base/retrieve` for one smartwatch file returned about 22.9 KB of content.
- A live call to `/knowledge-base/retrieve` for one product directory returned about 86.5 KB of content across 5 files.
- A live call to `/knowledge-base/retrieve` for `/` returned about 449 KB of content across 25 files.
- A live call to `/chat` with the shipped Google provider config streamed back an error content chunk plus `done`, not a server crash.
- In the shipped `.env`, all API key fields are still placeholders.
- Runtime config validation showed that `KNOWLEDGE_BASE_PATH` and `KNOWLEDGE_BASE_CHUNKS_PATH` do not affect `Settings()`, while exporting `KNOWLEDGE_BASE` and `KNOWLEDGE_BASE_CHUNKS` does.
- A local OpenAI-compatible stub provider on a test port was enough to run `/chat` end-to-end without changing project source files.
- In function mode, asking `What are all the technical specifications of SW-2100?` produced a clean event sequence: `tool_calls` for `Product-Line-A-Smartwatch-Series/SW-2100-Flagship.md`, then a `tool_results` payload of 22961 characters, then final answer content chunks, then `done`.
- In ReAct mode on the same query, the first streamed content chunks contained `<|Thought|>`, `<|Action|>`, and `<|Action Input|>` markers before the backend emitted a structured `tool_calls` event.
- In ReAct mode on the same query, the backend then emitted a `tool_results` payload of 22961 characters and a final answer whose streamed content still contained `<|Thought|>` and `<|Final Answer|>` markers before `done`.
- Backend debug logs in the ReAct run showed that the second provider call included both the assistant's prior action text and a synthetic user message beginning with `<|Observation|>` followed by the full retrieved file content and `Continue with your reasoning.`
- `Knowledge-Base-File-Summary/generate.py` only collects `*.md` files, summarizes them with an LLM, copies small files into the chunks mirror, and writes large-file chunks into subdirectories under `settings.knowledge_base_chunks`.
- The summary generator caches per-file results in `Knowledge-Base-File-Summary/summary_demo.json`.
- `Knowledge-Base-File-Summary/Summary Prompt.md` exists as a readable prompt reference, but the generator uses inline prompt strings from `generate.py` rather than importing that markdown file.
- The generator prints the full summarization prompt, including source file content, to stdout during processing.
- `backend/knowledge_base.py` reads explicit file paths directly, but directory retrieval and `/` retrieval enumerate only `*.md` files with `rglob("*.md")`.
- In `backend/knowledge_base.py`, the `file_path == "/"` branch comes after `full_path.is_dir()`, so when the base path exists the root case is effectively handled by the generic directory branch instead of its own special branch.
- The frontend loads providers and default provider/model into state, but `App.tsx` does not render any selector UI for either field.
- The frontend still sends `provider` and `model` back on `/chat`, even though backend handling currently ignores `request.model`.
- `frontend/src/components/ConfigPanel.tsx` calls `http://localhost:8000/api/config` directly instead of reusing the shared `API_BASE_URL` abstraction from `frontend/src/api.ts`.
- The frontend config panel therefore assumes a localhost backend even though the rest of the frontend is written against a configurable `VITE_API_BASE_URL`.
- The shipped frontend stream parser buffers ReAct content until it sees `<|Final Answer|>`, extracts only the text after that tag, and resets buffered thought/action text after `tool_results`.
- `frontend/src/components/ChatMessage.tsx` can render and copy full tool results, and only truncates on-screen display after 500000 characters.
- `backend/main.py` saves `.env`, calls `load_dotenv(override=True)`, and then reassigns `settings = Settings()` only inside `backend.main`.
- `backend/knowledge_base.py` constructs `knowledge_base = KnowledgeBase()` as a module-level singleton at import time from `backend.config.settings`.
- `backend/llm_provider.py` reads `settings.temperature` and `settings.max_tokens` from `backend.config.settings` when constructing provider payloads.
- The frontend config panel tells the user that configuration is applied immediately after save.

## Inferences

- The main innovation is not a new retrieval algorithm but a different retrieval contract: let the model navigate a human-readable knowledge map and pull full files on demand.
- This is a tool-augmented RAG system, but it is not a classic vector-store RAG pipeline.
- The system's multi-turn memory is mostly client-driven conversation replay plus in-loop tool observations, not a richer server-side session memory layer.
- The ReAct mode is more agentic in surface form, but both modes ultimately depend on the same single retrieval tool and file-backed knowledge store.
- The documentation/config mismatch around knowledge-base path variables is real, not just suspected: the documented `_PATH` names are ignored by the current settings model.
- The model-selection UX in the frontend appears weaker than advertised because the backend currently ignores `request.model`.
- The retrieval contract is operationally proven up to the LLM boundary: app boot, summary loading, prompt generation, and raw retrieval all work in a live backend run.
- The orchestration contract is now operationally proven beyond the retrieval boundary as well, as long as the provider speaks the expected OpenAI-compatible streaming format.
- The most immediate runtime risk is not backend instability but observation size: directory and full-corpus retrieval can become very large before any second-stage compression happens.
- Provider failures are handled gracefully enough for UX continuity because `/chat` streams the error text back as content instead of exploding the request.
- Function mode has the cleaner frontend contract because tool intent appears as a structured event before the final answer text arrives.
- ReAct mode is still more format-fragile than function mode, but the shipped UI hides some of that raw SSE noise by filtering on `<|Final Answer|>`, so the main issue is tight frontend/backend protocol coupling rather than unavoidable tag leakage to end users.
- The "large observation" risk is now more concrete in ReAct mode too, because the full retrieved payload is re-injected as a synthetic user message before the next provider call.
- Preprocessing is more operationally central than the README wording suggests: the architecture is designed around a mirrored retrieval corpus and summary artifacts, not just an optional helper script.
- The README and frontend imply more runtime configurability than the code currently exposes: provider/model values are loaded and echoed, but there is no visible selector UI and the config editor is not deployment-agnostic.
- The effective runtime surface is narrower than the declared product surface:
  path configuration, provider/model selection, and frontend deployment assumptions all drift from the README's more flexible framing.
- Retrieval semantics are narrower than the top-level README framing suggests, because the preprocessing and directory traversal paths are markdown-centric.
- The config editor's hot-reload story appears only partially true by static inspection: rebinding `settings` inside `backend.main` does not recreate imported singletons or refresh other modules that still reference `backend.config.settings`.
- This likely creates a split config surface where some values refresh immediately:
  provider lists and provider-specific credentials read through `os.environ`
  while others risk staying stale until process restart:
  knowledge-base paths, summary paths, temperature, and max-token settings

## Open Questions

- In practice, does the file-summary map reduce retrieval steps enough to outperform a simpler chunk-search baseline?
- Does returning full directories or `/` create context blow-up on real queries?
- How large can retrieval payloads get before answer quality or latency degrades materially in a real provider-backed run?
- How large can retrieval payloads get before answer quality or latency degrades materially in either orchestration mode?
- Is the frontend model selector intentionally cosmetic, or is backend support incomplete?
- How robust is the ReAct parser when the model deviates from the expected `<|...|>` format?
- Should the frontend or backend strip ReAct protocol markers before displaying the final answer?
- Should the config panel use the same API base URL abstraction as the rest of the frontend?
- Should the README explicitly document that preprocessing and bulk retrieval are markdown-oriented rather than file-type-agnostic?
- Should the summary generator stop printing full source content into logs during summarization?
- Should `/api/config` update shared runtime state in one place and recreate settings-dependent singletons so "save and apply" actually matches behavior?

## Improvement Ideas

- Gate or remove the raw `.env` read/write surface from the default app path, or at minimum restrict it to trusted local-only usage.
- Centralize config hot reload in one shared runtime settings layer and recreate settings-dependent singletons after a save.
- Make function-mode tool-call accumulation either support multiple tool calls correctly or explicitly reject multi-call streams.
- Unify frontend config requests behind the shared `API_BASE_URL` abstraction so deployment assumptions are consistent.

## Review-Grade Candidates

- High: `backend/main.py` exposes unauthenticated `.env` read/write endpoints at `/api/config`, while the app also enables wildcard CORS and the default launcher binds the backend to `0.0.0.0`.
- High: config hot reload is only partially applied because `backend/main.py` rebinds its own `settings`, but `backend/knowledge_base.py` and `backend/llm_provider.py` still depend on earlier module-level state.
- Medium: function-mode streaming currently accumulates only one tool call correctly; multiple tool calls in one assistant turn can be merged into a single corrupted payload.
- Medium: `frontend/src/components/ConfigPanel.tsx` hardcodes `http://localhost:8000`, so the config editor can break whenever the frontend is pointed at a non-local backend through `VITE_API_BASE_URL`.

## Blocked or Low-Confidence Record

- Files Checked: `backend/main.py`, `backend/prompts.py`, `backend/react_handler.py`, `backend/knowledge_base.py`, `backend/llm_provider.py`, `backend/config.py`, `backend/models.py`, `Knowledge-Base-File-Summary/generate.py`, `Knowledge-Base-File-Summary/Summary Prompt.md`, `frontend/src/App.tsx`, `frontend/src/api.ts`, `frontend/src/types.ts`, `README.md`, `.env.example`
- Current Blocker: real third-party provider validation is still blocked by placeholder API credentials in the shipped `.env`, even though end-to-end chat has now been validated with a local compatible stub
- Working Hypothesis: this is a knowledge-map-driven retrieval agent with working dual-mode orchestration, and the main remaining risks are oversized retrieval payloads, preprocessing/config drift, partial hot-reload semantics, and frontend/backend contract friction rather than failure to boot
- Resume From: continue the static audit around config propagation, docs drift, and frontend/backend contract mismatches, then only return to real-provider validation if it becomes necessary later
