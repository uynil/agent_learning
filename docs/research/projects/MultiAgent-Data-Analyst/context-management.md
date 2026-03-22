---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: resolve the MemoryTools API mismatch and rerun the resume paths
---

# Context Layers

This project uses several different kinds of context, not one unified memory system:

- `st.session_state` for page-level UI handoff
- `MemoryTools` for JSON key-value storage
- `MemoryAgent` for append-only history
- `A2ABus` for message passing and audit logs
- filesystem artifacts such as reports, visualizations, and model files
- direct Python arguments passed between agent objects

# State Handoffs

- `streamlit_app/pages/Dataset_Explorer.py` writes dataset summaries and profiles into memory.
- `streamlit_app/pages/AutoML.py` stores `model_output` in `st.session_state`.
- `streamlit_app/pages/Verifier.py` and `streamlit_app/pages/Notebook_Report.py` try to recover previous outputs from session state or memory.
- `src/orchestrator.py` expects `profiler_output`, `eda_output`, `model_output`, and `verifier_output` to exist before notebook synthesis.
- `src/agents/notebook_synthesizer_agent.py` reads the prior outputs and writes the notebook path back to memory.

# Persistence and Memory

- `src/tools/memory_tools.py` stores one JSON object per key in `streamlit_app_storage/memory/memory.json`.
- `src/agents/memory_agent.py` stores an append-only `history` array in `memory/project_memory.json`.
- `src/core/a2a_bus.py` tries to persist message queues through `MemoryTools`.
- `src/tools/a2a_tools.py` keeps messages only in memory during the session.
- `mcp/manifest.json` suggests read/write/list tools for memory, but the repository code does not expose a single clean memory abstraction.

# Resumability

- The design clearly wants resumability.
- The notebook step can reconstruct a report from stored outputs.
- The Streamlit pages are written to use session state first and memory as a fallback.
- The A2A bus is intended to make inter-agent coordination inspectable.
- However, the current implementation has a real API mismatch: several call sites use `MemoryTools.load()` with no argument, but the implementation requires a key.

# Current Problems

- `src/orchestrator.py` calls `memory.load().get(...)`, which does not match `MemoryTools.load(self, key)`.
- `src/core/a2a_bus.py` uses the same invalid no-argument `load()` call in its persistence path.
- `streamlit_app/pages/Verifier.py` and `Notebook_Report.py` also call `memory.load()` with no argument.
- Because of that mismatch, the intended recovery-from-memory flow is likely broken.
- `ModelAgent` is also incomplete, so the context handoff from EDA to model is not fully reliable yet.
- Gemini key naming is also inconsistent between `src/tools/notebook_tools.py` and `streamlit_app/pages/AutoML.py`, which makes notebook synthesis and model explanation depend on different env var conventions.

# Context Management Summary

- The project manages context mostly by moving structured artifacts forward, not by maintaining a long conversation state.
- That is a reasonable pattern for data analysis, but it needs one consistent persistence API.
- Right now the repo has the right idea and the wrong glue.

# Open Questions

- Should `MemoryTools.load()` be changed to return all memory, or should every caller be updated to pass a key?
- Is `MemoryAgent` intended to replace `MemoryTools`, or is it legacy?
- Should `A2ABus` persist to the same store as the Streamlit memory pages?
- Is the notebook synthesis step supposed to be the canonical resumable checkpoint?
