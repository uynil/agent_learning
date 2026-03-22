---
project: deep-rag
source_path: examples/deep-rag
status: first-pass complete
confidence: medium
last_updated: 2026-03-22
next_action: if needed, turn the severity-ranked review-grade candidates into formal code-review findings or a concrete patch plan
---

# Notes

## Session Notes

- The project's core idea is clearer than its name might suggest: the innovation is a knowledge-map summary plus active retrieval, not a hidden vector trick.
- It is more agentic than a static RAG chain because the model chooses retrieval paths iteratively.
- It is less "deep research" than report-oriented research agents because it lacks explicit decomposition and round summaries.
- A backend-only live run is now complete:
  - backend booted in a local venv
  - health/config/summary/prompt/retrieval endpoints worked
  - `/chat` reached the provider boundary and failed cleanly because the shipped API keys are placeholders
- The retrieval-size risk is real in runtime, not just in code reading:
  - single file retrieval: about 22.9 KB
  - one directory retrieval: about 86.5 KB
  - full corpus retrieval: about 449 KB
- The path-variable mismatch is also real in runtime:
  - `KNOWLEDGE_BASE_PATH` and `KNOWLEDGE_BASE_CHUNKS_PATH` do not change the loaded settings
  - `KNOWLEDGE_BASE` and `KNOWLEDGE_BASE_CHUNKS` do
- A local OpenAI-compatible stub provider was enough to prove that `/chat` is not fundamentally blocked:
  - function mode completed with `tool_calls -> tool_results -> answer -> done`
  - ReAct mode completed with streamed action markers, then `tool_calls`, then `tool_results`, then a final answer
- Static frontend review refined the ReAct conclusion:
  - raw SSE chunks do contain protocol markers
  - but the shipped frontend buffers them and only renders text after `<|Final Answer|>`
  - so the bigger issue is protocol coupling, not guaranteed end-user leakage
- The frontend contract is thinner than it first looked:
  - provider/model values are loaded into state but there is no selector UI
  - the config editor hardcodes `http://localhost:8000` instead of using the shared API base URL
- The config editor story is also thinner than it looks:
  - `main.py` rebinds `settings` locally after saving `.env`
  - but `knowledge_base` is a module-level singleton built earlier from `backend.config.settings`
  - and `LLMProvider` still reads temperature/max-token fields from `backend.config.settings`
  - so "save and apply" looks only partially reliable by static inspection
- The preprocessing path is stronger than the README framing suggests:
  - the generator only processes markdown
  - it mirrors small files into the retrieval corpus
  - it writes large files as chunk subdirectories
  - and it logs full prompts, including source content, during summarization
- The static contract audit is now effectively complete:
  README env names, settings field names, frontend config routing, and visible runtime controls have all been reconciled enough for a first-pass conclusion.

## What I Understood

- The preprocessing stage is first-class here; runtime quality depends heavily on the summary generator.
- The single `retrieve_files` tool is doing most of the system-level work.
- Function-calling mode and ReAct mode share the same retrieval contract and differ mostly in protocol shape.
- The protocol-shape difference is not cosmetic, but the visible UX depends on client-side cleanup logic rather than the backend alone.
- The system promises more runtime flexibility than it currently surfaces:
  the README and frontend state suggest configurable provider/model/path behavior, but the effective launch surface is narrower.
- The running configuration is not backed by one authoritative live settings object once `.env` edits happen inside the app.

## What I Changed My Mind About

- I initially expected a more complicated multi-tool agent.
- After reading the code, the main novelty is not tool variety but knowledge navigation over preserved structure.
- I expected ReAct mode to feel roughly equivalent to function mode at the UI boundary.
- After reading the frontend, I no longer think the default UI simply dumps raw ReAct scaffolding to the user; instead, it hides part of that scaffolding with custom parsing, which creates a different kind of fragility.

## Worth Revisiting

- Whether real providers preserve the same function-mode behavior and the same frontend-cleanable ReAct marker pattern seen with the local compatible stub.
- Whether the frontend model selector is functionally incomplete.
- Whether large retrieval payloads need compression before re-injection.
- Whether the summary generator should stop printing source content into logs.

## What Now Looks Review-Grade

- High: unauthenticated config-file exposure and mutation.
  `backend/main.py` exposes raw `.env` read/write endpoints at lines 55-93, `backend/main.py` also enables wildcard CORS at lines 26-32, and `start.sh` binds uvicorn to `0.0.0.0` at lines 60-65.
  This is a concrete security issue if the app is exposed beyond a tightly trusted local machine.
- High: partial hot reload behind an "applies immediately" UI.
  `backend/main.py` only rebinds its local `settings` at lines 83-89, but `backend/knowledge_base.py` creates `knowledge_base = KnowledgeBase()` from earlier settings at line 97 and `backend/llm_provider.py` still reads `settings.temperature` and `settings.max_tokens` at lines 36-48.
  The resulting behavior is likely split-brain configuration rather than true immediate application.
- Medium: function-mode tool-call aggregation is not safe for multiple tool calls.
  In `backend/main.py` lines 193-210, the stream parser stores only one `accumulated_tool_call` and appends later argument fragments into that single structure.
  If a provider emits more than one tool call in the same turn, the second call can be collapsed into the first call's arguments.
- Medium: frontend config editor ignores the shared API base abstraction.
  `frontend/src/components/ConfigPanel.tsx` hardcodes `http://localhost:8000` at lines 24 and 43 even though `frontend/src/api.ts` defines `VITE_API_BASE_URL` as the shared base URL at lines 3-21.
  This makes the config editor fail in deployments where chat and info requests succeed through a non-default backend URL.

## Severity Ordering Rationale

- High:
  the `.env` surface can expose secrets and mutate runtime credentials directly, and the hot-reload issue can make the app misrepresent which configuration is actually active.
- Medium:
  the tool-call aggregation issue depends on a provider emitting multiple tool calls in one turn, and the config-panel routing bug is deployment-specific rather than universal.

## What Still Belongs In Research Notes

- The markdown-only retrieval surface is an architectural boundary and product-shape observation, but not automatically a code defect.
- The mirrored markdown corpus introduces freshness risk, but whether that becomes a bug depends on actual project maintenance expectations.
- The difference between `Summary Prompt.md` and the executed generator prompt is important for research accuracy, but not necessarily a runtime defect by itself.
- Whether the config editor should be treated as a dev convenience rather than a trustworthy live-reconfiguration surface.

## Linked Comparisons

- `docs/research/comparisons/context-management-patterns.md`
- `docs/research/comparisons/deep-research-patterns.md`
- `docs/research/comparisons/prompt-function-mapping-patterns.md`
- `docs/research/comparisons/prompt-organization-patterns.md`

## Blocked or Low-Confidence Record

- Status: first-pass complete, high-confidence on backend retrieval plus local compatible-provider chat runtime, medium-confidence on real vendor-backed chat runtime
- Files Checked: `backend/main.py`, `backend/prompts.py`, `backend/react_handler.py`, `backend/knowledge_base.py`, `backend/llm_provider.py`, `backend/config.py`, `backend/models.py`, `Knowledge-Base-File-Summary/generate.py`, `Knowledge-Base-File-Summary/Summary Prompt.md`, `frontend/src/App.tsx`, `frontend/src/api.ts`, `frontend/src/types.ts`, `README.md`, `.env.example`, `requirements.txt`, `start.sh`
- Blocker: provider credentials are placeholders, so real vendor-backed answer generation still cannot complete meaningfully from the shipped config
- Working Hypothesis: this is active file-navigation RAG with a healthy dual-mode runtime, but its main product risks are config mismatch, partial hot-reload semantics, uncompressed retrieval payloads, preprocessing drift, and frontend/backend contract fragility
- Resume From: compare against other local RAG systems and only revisit real-provider testing if a new question depends on it

## Next Research Plan

1. If needed, turn the top four review-grade candidates into formal code-review findings with severity ordering.
2. Treat hot-reload semantics as a first-class comparison dimension for local RAG runtimes.
3. Extend the same retrieval-topology lens to one more non-file-navigation system if needed.
4. Revisit real-provider validation only if a later comparison needs it.
