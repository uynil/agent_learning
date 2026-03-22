---
project: deep-rag
source_path: examples/deep-rag
status: first-pass complete
confidence: medium
last_updated: 2026-03-21
next_action: compare this knowledge-map retrieval shape with graph-heavy RAG systems and only revisit real-provider validation if a new question requires it
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

## Linked Comparisons

- `docs/research/comparisons/context-management-patterns.md`
- `docs/research/comparisons/deep-research-patterns.md`
- `docs/research/comparisons/prompt-function-mapping-patterns.md`
- `docs/research/comparisons/prompt-organization-patterns.md`

## Blocked or Low-Confidence Record

- Status: first-pass complete, high-confidence on backend retrieval plus local compatible-provider chat runtime, medium-confidence on real vendor-backed chat runtime
- Files Checked: `backend/main.py`, `backend/prompts.py`, `backend/react_handler.py`, `backend/knowledge_base.py`, `backend/llm_provider.py`, `backend/config.py`, `backend/models.py`, `Knowledge-Base-File-Summary/generate.py`, `Knowledge-Base-File-Summary/Summary Prompt.md`, `frontend/src/App.tsx`, `frontend/src/api.ts`, `frontend/src/types.ts`, `README.md`, `.env.example`, `requirements.txt`, `start.sh`
- Blocker: provider credentials are placeholders, so real vendor-backed answer generation still cannot complete meaningfully from the shipped config
- Working Hypothesis: this is active file-navigation RAG with a healthy dual-mode runtime, but its main product risks are config mismatch, uncompressed retrieval payloads, preprocessing drift, and frontend/backend contract fragility
- Resume From: compare against other local RAG systems and only revisit real-provider testing if a new question depends on it

## Next Research Plan

1. Compare `deep-rag` against graph-heavy systems like `LightRAG` to sharpen the boundary between knowledge-map navigation and graph-centric retrieval.
2. Decide whether the remaining config/UI drift deserves code review findings or should stay as research notes only.
3. Revisit real-provider validation only if a later comparison needs it.
