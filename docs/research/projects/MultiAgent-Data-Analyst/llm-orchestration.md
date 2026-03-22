---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: validate one real request end-to-end and record where the LLM boundaries actually are
---

# LLM Call Map

Observed LLM calls are concentrated at the edge of the workflow, not in the core agent handoff path.

1. `src/tools/notebook_tools.py`
   - `_generate_gemini_insights(...)`
   - turns EDA JSON and model JSON into narrative report sections
2. `streamlit_app/pages/AutoML.py`
   - `gemini_explain_model(...)`
   - turns model-result JSON into beginner-friendly natural-language explanation

The main profiler -> EDA -> model -> verifier -> notebook pipeline itself is otherwise deterministic Python and sklearn logic.

# Scheduling Pattern

- There is no top-level LLM planner or tool-routing agent in the observed code.
- Scheduling is mostly `data artifact -> deterministic step -> persist output -> notify next stage`.
- The LLM appears after the core analytics are done, mainly to explain results for humans.
- This makes the project more of an event-driven workflow with LLM augmentation than an LLM-orchestrated agent system.

# Structured Output Strategy

- The core pipeline uses Python dicts, pandas objects, sklearn artifacts, and JSON files.
- Gemini outputs are free-form narrative text, not schema-constrained machine outputs.
- Notebook synthesis embeds prior structured artifacts into a notebook, then appends a Gemini-written narrative section.

# Reliability Mechanisms

- Most reliability comes from deterministic local code, not from prompt constraints.
- `VerifierAgent` assigns a simple quality label to the trained model output.
- Logging exists in some agents, but it is not consistently unified across the repo.
- Gemini calls are wrapped in `try/except` and degrade to error text if the API call fails.

# Context Passing

- Agent-to-agent context is passed mostly as Python objects, saved JSON artifacts, and bus messages.
- Gemini calls receive already-computed JSON summaries rather than raw datasets.
- The notebook synthesis step is effectively a final evidence-compression stage.
- In the Streamlit path, `st.session_state` is another layer of transient context.

# Open Questions

- Does the repository actually execute the full agent chain cleanly once the missing model and memory pieces are repaired?
- Is the Gemini-based notebook insight step a core stage or an optional enhancement?
- Are there any additional LLM paths outside `src/` and `streamlit_app/` that are not active in the current code tree?
