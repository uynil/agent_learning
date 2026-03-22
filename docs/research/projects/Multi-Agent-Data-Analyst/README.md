---
project: Multi-Agent-Data-Analyst
source_path: examples/Multi-Agent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: install runtime dependencies and run one real SQL and one DataFrame prompt
---

# Multi-Agent-Data-Analyst

## Project Summary

`Multi-Agent-Data-Analyst` is a data-analysis assistant built around one top-level orchestrator and two specialized execution agents. The orchestrator decides whether a request should go through SQL-backed analysis or direct DataFrame visualization, then delegates to a SQL analysis pipeline or a Plotly visualization pipeline. In practice, this is closer to a route-and-delegate analysis workbench than a multi-agent negotiation system.

## Quick Classification

- Style: orchestrator + specialized sub-agents
- Best fit: analysis routing, SQL generation, and chart generation
- Deep research behavior: weak; there is no explicit query rewrite, subquestion decomposition, or reviewer loop beyond execution retries
- Prompt layer: inline and code-coupled, not centralized in a prompt library

## Key Files

- `examples/Multi-Agent-Data-Analyst/main.py`
- `examples/Multi-Agent-Data-Analyst/app/agent_orchestrator.py`
- `examples/Multi-Agent-Data-Analyst/app/tools/sql_data_analyst_agent.py`
- `examples/Multi-Agent-Data-Analyst/app/tools/data_analyst_agent.py`
- `examples/Multi-Agent-Data-Analyst/app/visualization_server.py`
- `examples/Multi-Agent-Data-Analyst/tests/test_agent_orchestrator.py`
- `examples/Multi-Agent-Data-Analyst/tests/test_sql_data_analyst_agent.py`
- `examples/Multi-Agent-Data-Analyst/tests/test_data_analyst_agent.py`
- `examples/Multi-Agent-Data-Analyst/tests/test_main.py`

## Research Notes

- [architecture.md](architecture.md)
- [llm-orchestration.md](llm-orchestration.md)
- [context-management.md](context-management.md)
- [deep-research-mode.md](deep-research-mode.md)
- [prompts-analysis.md](prompts-analysis.md)
- [notes.md](notes.md)

## Observed Facts

- `main.py` launches a Gradio UI when run without CLI args and binds it to port 8080.
- The CLI path supports `--mode dataframe|sql|auto`, file upload, and DB URL.
- `agent_orchestrator.py` defines a PydanticAI `Agent` with `result_type=AnalysisResult`.
- The orchestrator exposes three tools: `sql_agent`, `visualization_agent`, and `determine_data_source`.
- `sql_agent` instantiates `SQLDataAnalysisAgent`, then writes Plotly output to `visualizations/analysis_result.html` if present.
- `visualization_agent` instantiates `DataVisualizationAgent` and returns generated code plus explanation.
- `SQLDataAnalysisAgent` uses two LLM calls even before visualization: one for SQL text and one for a Python executor function, then a third LLM call for charting when visualization is needed.
- `DataVisualizationAgent` summarizes the DataFrame, asks for JSON `{code, explanation}`, then executes the generated code with retries.
- Error handling is mostly local dict-return patterns plus `try/except`; there is no persisted session memory.
- `requirements.txt` and `pyproject.toml` include a large template-like dependency set, including FastAPI, Gradio, PydanticAI, LangGraph, Plotly, SQLAlchemy, OpenAI, and others.
- `app/rag/vector_store.py` exists, but I found no call site in the main path.

## Inferences

- The system is not a coordinated multi-agent society; it is one top-level router plus two task-specialized executors.
- The SQL path is effectively a three-stage LLM/code synthesis pipeline: generate SQL, generate an execution wrapper, optionally generate visualization code.
- The prompt strategy is intentionally lightweight but tightly coupled to runtime execution, so prompt quality and code safety are entangled.
- The README's FastAPI-oriented setup looks stale relative to the actual Gradio entrypoint in `main.py`.
- `app/rag/vector_store.py` is probably auxiliary or leftover rather than part of the observed production path.

## Open Questions

- Is `app/rag/vector_store.py` used anywhere outside the current tree, or is it dead/experimental code?
- Is the README's mention of API access at `localhost:8080` supposed to describe the Gradio UI, or is an API layer missing from the current checkout?
- Does `determine_data_source` materially influence routing at runtime, or is it mostly a convenience helper?
- How often do the generated SQL executor/function fallbacks and visualization retries actually rescue bad model output in practice?

## Next Action

- Install dependencies, set the needed API key(s), and run one real SQL prompt and one DataFrame prompt to verify whether the observed route-and-delegate behavior matches the static reading.
