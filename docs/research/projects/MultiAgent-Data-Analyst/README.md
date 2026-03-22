---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: verify the Streamlit/src path end-to-end and fix the memory/load mismatches before a live run
---

# MultiAgent-Data-Analyst

## Project Summary

`MultiAgent-Data-Analyst` is an end-to-end data-analysis demo built around a staged pipeline: dataset profiling, EDA, model training, verification, notebook synthesis, and Gemini-generated explanations. The observed implementation is centered on `src/` agents, `streamlit_app/` pages, JSON memory, and an A2A-style message bus. It is better understood as a workflow-oriented analytics demo than as a tightly orchestrated LLM-first agent runtime.

## Quick Classification

- Style: staged multi-agent analytics workflow with Streamlit and A2A messaging
- Best fit: profiler -> EDA -> model -> verifier -> notebook handoff automation
- LLM usage: narrow and edge-oriented; mainly Gemini narrative synthesis rather than core orchestration
- Deep research behavior: absent; this is a data pipeline, not a question-decomposition research agent

## Key Files

- `examples/MultiAgent-Data-Analyst/src/orchestrator.py`
- `examples/MultiAgent-Data-Analyst/src/agents/profiler_agent.py`
- `examples/MultiAgent-Data-Analyst/src/agents/eda_agent.py`
- `examples/MultiAgent-Data-Analyst/src/agents/model_agent.py`
- `examples/MultiAgent-Data-Analyst/src/agents/verifier_agent.py`
- `examples/MultiAgent-Data-Analyst/src/agents/notebook_synthesizer_agent.py`
- `examples/MultiAgent-Data-Analyst/src/core/a2a_bus.py`
- `examples/MultiAgent-Data-Analyst/src/tools/memory_tools.py`
- `examples/MultiAgent-Data-Analyst/src/tools/notebook_tools.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/app.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/pages/Dataset_Explorer.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/pages/EDA_Dashboard.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/pages/AutoML.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/pages/Verifier.py`
- `examples/MultiAgent-Data-Analyst/streamlit_app/pages/Notebook_Report.py`
- `examples/MultiAgent-Data-Analyst/mcp/manifest.json`

## Research Notes

- [architecture.md](architecture.md)
- [llm-orchestration.md](llm-orchestration.md)
- [prompts-analysis.md](prompts-analysis.md)
- [context-management.md](context-management.md)
- [notes.md](notes.md)

## Observed Facts

- `src/orchestrator.py` wires a profiler -> EDA -> model -> verifier -> notebook pipeline through `A2ABus` and `MemoryTools`.
- `src/agents/notebook_synthesizer_agent.py` persists notebook output and can trigger on a `verifier.completed` message.
- `src/tools/notebook_tools.py` uses Gemini to generate notebook insights from EDA and model JSON.
- `streamlit_app/pages/*` stores outputs in `st.session_state` and also tries to recover them from JSON memory files.
- `mcp/manifest.json` exposes a tool surface for file, dataset, model, notebook, job, and memory operations.
- `src/agents/model_agent.py` is a wrapper that expects `_train_and_evaluate(...)`, but that method is not defined in the file.
- Several resume/fallback paths call `MemoryTools.load()` with no argument, even though `src/tools/memory_tools.py` defines `load(self, key)`.
- Gemini-related environment handling is inconsistent: `src/tools/notebook_tools.py` reads `GOOGLE_API_KEY`, while `streamlit_app/pages/AutoML.py` configures `GEMINI_API_KEY`.
- A search across `src/` and `streamlit_app/` found Gemini calls in `src/tools/notebook_tools.py` and `streamlit_app/pages/AutoML.py`, but no `pydantic_ai`, `ChatOpenAI`, or `agent_orchestrator.py` in this repository.

## Inferences

- The repository has one main source lineage, but it is exposed through multiple surfaces: a scriptable `src/` pipeline, Streamlit pages, and MCP tool metadata.
- Most of the system is deterministic analytics tooling with a small LLM surface for notebook synthesis and plain-language explanation.
- The memory layer is intended to provide resumability, but the current API mismatches make that goal fragile.
- The project’s “multi-agent” framing is stronger at the naming and UI level than at the autonomy level.

## Open Questions

- Which entry path is canonical: `src/orchestrator.py`, the Streamlit app, or the MCP-backed tool surface?
- Is `ModelAgent._train_and_evaluate()` implemented elsewhere, or is the current file genuinely incomplete?
- Are the `MemoryTools.load()` no-arg calls stale code or a missing API migration?
- Which Gemini key name is canonical: `GOOGLE_API_KEY` or `GEMINI_API_KEY`?
- Does the project run end-to-end without edits once dependencies and API keys are set?

## Next Action

- Pick one runtime entry as canonical, repair the memory API mismatches and the `ModelAgent` training placeholder, then run one live dataset through the end-to-end flow.
