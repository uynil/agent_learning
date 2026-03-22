---
project: Multi-Agent-Data-Analyst
source_path: examples/Multi-Agent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: run live queries and compare the runtime behavior with these first-pass notes
---

# Notes

## First-Pass Takeaways

- The project name implies "multi-agent", but the implementation is really an orchestrator plus specialized executor classes.
- The SQL path is the most interesting part: it generates SQL, generates a Python wrapper, executes the query, and optionally generates a chart.
- The visualization path is simpler: summarize DataFrame -> ask for JSON -> execute Plotly code -> retry if needed.
- The prompt logic is deeply embedded in runtime code instead of being centralized in a separate prompt library.
- The UI is more Gradio-first than the README suggests.

## Things Worth Remembering

- `process_user_input_stream` is a status wrapper, not true token streaming.
- `usage_limits` is only clearly enforced at the orchestrator boundary.
- The generated code paths rely on `exec()`, so the project trades safety for flexibility.
- `app/rag/vector_store.py` may be an experimental branch or leftover module rather than a main-path dependency.

## Mismatches / Stale Signals

- The README appears to carry FastAPI/template-era language, but the actual `main.py` entrypoint is a Gradio app plus CLI.
- The package metadata in `pyproject.toml` also looks template-derived, which suggests the repository may have accumulated some carried-over scaffolding.

## Follow-Up Ideas

- Run one real SQL prompt against a known SQLite DB and inspect the generated SQL, wrapper, and chart output.
- Run one real DataFrame prompt and compare the generated visualization code against the DataFrame summary.
- Decide whether `determine_data_source` deserves to be elevated into a real routing policy or left as a helper.

