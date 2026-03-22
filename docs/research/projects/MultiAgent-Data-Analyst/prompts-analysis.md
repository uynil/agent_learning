---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare the embedded prompts against a live run and see which instructions actually matter
---

# Prompt Inventory

- `src/tools/notebook_tools.py`
  - Gemini prompt: write professional ML-report insights from EDA and model JSON.
- `streamlit_app/pages/AutoML.py`
  - Gemini prompt: explain model quality, performance, and next steps in plain language.

## Non-LLM contracts

- `mcp/manifest.json` is not a prompt, but it acts like a tool contract for the same system surface.
- A2A message topics such as `profiler.completed`, `model.trained`, and `verifier.completed` also function as coordination contracts.

# Prompt Organization Pattern

- Prompts are embedded directly in leaf helpers rather than stored in a central prompt library.
- Most of the repo does not use prompts at all; it uses deterministic analytics code.
- The prompt layer is organized around narrative synthesis and explanation rather than around routing or executable-code generation.
- Prompting is a small supporting layer, not the architectural backbone.

# Prompt Structure

- Role statement: “senior ML engineer” or “ML expert”.
- Input framing: embed EDA/model result JSON directly into the prompt.
- Requested sections: findings, model strengths/weaknesses, next steps, or beginner-friendly explanation.
- Tone/audience control: one prompt is more professional-report oriented, the other is more beginner oriented.
- There is no schema-driven parser contract in the observed prompts.

# Function-to-Mechanism Mapping

- `NotebookTools` prompt
  - Function: convert structured EDA/model outputs into a notebook-ready narrative section
  - Mechanism: inject JSON artifacts and ask for fixed categories such as findings, strengths, weaknesses, and next steps
- `AutoML.py` prompt
  - Function: explain model performance to a beginner
  - Mechanism: inject result JSON and ask explicit questions about task type, quality, and improvements
- A2A/event contracts
  - Function: move work between agents without a language-model planner
  - Mechanism: explicit topic names and persisted artifacts rather than prompt reasoning

# Prompt-to-Code Coupling

- The coupling is relatively loose compared with code-generation-heavy agent projects.
- Prompt output is displayed to humans or embedded into a notebook; it is not executed as code.
- That makes prompt quality important for clarity, but less central to runtime safety.

# Constraint and Reliability Strategy

- Keep prompts narrow and post-hoc: explain already-computed artifacts rather than decide the workflow.
- Use explicit sub-questions to control the narrative shape.
- Fall back to an error string when Gemini calls fail.
- Let deterministic code produce the evidence first, then let the model summarize it.

# Likely Failure Modes

- Gemini outputs may become generic because there is no strict output schema.
- Inconsistent env var names can make one prompt path work while another silently fails.
- Broken memory recovery can leave the notebook/explanation prompts with incomplete artifacts.
- Because prompts are sparse and distributed, they can drift quietly as the workflow evolves.

# Reusable Prompt Patterns

- Explain structured JSON after deterministic processing, rather than asking the model to do the computation itself.
- Use audience-specific framing to generate different explanation layers from the same artifact.
- Keep prompts local when the LLM is only a report writer, not the main planner.
