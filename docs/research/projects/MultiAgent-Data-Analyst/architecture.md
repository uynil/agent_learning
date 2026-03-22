---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: confirm which entry path is canonical and whether the Streamlit/A2A path runs cleanly end-to-end
---

# Architecture Summary

This repository is best described as one workflow-oriented analytics system with several entry surfaces:

1. `src/orchestrator.py`: a scriptable linear pipeline that wires the agents together.
2. `streamlit_app/*`: a UI shell that triggers the same analysis stages and surfaces stored artifacts.
3. `mcp/manifest.json`: a declarative tool inventory aligned with the project’s file/data/model/notebook operations.

The core architecture is not an LLM router. It is a staged analytics pipeline that uses agents as modular processing steps.

# Core Runtime Flow

## Main Components

- `src/orchestrator.py`
- `src/agents/profiler_agent.py`
- `src/agents/eda_agent.py`
- `src/agents/model_agent.py`
- `src/agents/verifier_agent.py`
- `src/agents/notebook_synthesizer_agent.py`
- `src/core/a2a_bus.py`
- `src/tools/*`
- `streamlit_app/pages/*`

## Flow

Dataset upload -> profiler -> EDA -> model training -> verifier -> notebook synthesis -> Gemini insights.

The Streamlit pages can also run the steps manually and cache outputs in `st.session_state` or JSON memory.

## Characteristics

- More like a workflow engine than a free-form agent system
- Most steps are deterministic analytics code, not LLM reasoning
- A2A messages are used as coordination events
- Notebook synthesis is the main LLM-adjacent step

# Module Breakdown

- `ProfilerAgent`: basic profiling and memory persistence.
- `EDAAgent`: descriptive stats, plots, correlations, outliers, VIF.
- `ModelAgent`: intended AutoML wrapper, but currently expects a missing `_train_and_evaluate()`.
- `VerifierAgent`: lightweight quality tagging over metrics.
- `NotebookSynthesizerAgent`: merges outputs into a notebook and saves it.
- `MemoryTools`: JSON key-value storage for outputs.
- `MemoryAgent`: append-only history store, but it is not the main memory API used elsewhere.
- `A2ABus` in `src/core/a2a_bus.py`: message queue and audit log with persistence intent.
- `A2A_GLOBAL_BUS` in `src/tools/a2a_tools.py`: a separate manual bus implementation used by the Streamlit dashboard.
- `mcp/manifest.json`: a declarative tool inventory that mirrors the project’s tool-first design.

# State and Data Flow

- The pipeline threads state through `st.session_state`, `MemoryTools`, A2A messages, and generated artifacts on disk.
- The notebook step expects four outputs to already exist: profiler, EDA, model, and verifier.
- The persistence story is fragmented: session memory, JSON memory, A2A inboxes, and artifact folders all coexist.

# Trade-offs

- The design is modular and easy to demo, but it spreads control across CLI/script, Streamlit pages, JSON memory, and bus state.
- The LLM usage is narrow and relatively safe because generated text is mostly explanatory, not executable.
- The memory model supports handoff and replay in principle, but the current implementation contains API mismatches.

# Open Questions

- Which entry path is the intended default for users?
- Is `src/orchestrator.py` a demo harness, or the real orchestration reference?
- Should the A2A bus and memory layers be unified?
- Should `MemoryAgent` and `MemoryTools` be unified into one persistence abstraction?
