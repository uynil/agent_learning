---
project: Multi-Agent-Data-Analyst
source_path: examples/Multi-Agent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: confirm with a live run that there is no deeper multi-turn research loop hidden behind the UI
---

# Deep Research Mode

## Current Classification

This project does not implement a true deep-research mode.

The strongest observed behavior is a single-request analysis flow with local retries:

- SQL generation can fall back from direct execution to a generated executor function.
- Visualization generation can retry when the first code attempt fails.
- The UI can stream status text, but not a real iterative research plan.

That is useful, but it is still far from a deep-research loop.

## What Is Missing for Deep Research

- No query rewrite stage.
- No subquestion decomposition.
- No planner/executor/reviewer separation.
- No evidence-accumulation loop across many turns.
- No sectioned research report synthesis.
- No explicit stopping criterion for "enough research has been done".

## Closest Approximation

The SQL path has a small internal refinement loop:

1. generate SQL
2. generate a Python wrapper for that SQL
3. try direct execution
4. fall back to the generated wrapper
5. optionally generate a chart

The DataFrame path has a similarly small internal refinement loop:

1. summarize the data
2. generate JSON code
3. execute the code
4. retry if the generated code fails

These are execution retries, not research decomposition.

## Inferences

- The project is built for task completion, not exploratory multi-hop research.
- If a user asks for "deep research", the current system would likely need an additional planner layer rather than only better prompts.
- The UI chat history could support a future deep-research workflow, but the backend does not currently exploit it.

## Open Questions

- Would a query-rewrite or question-decomposition layer be useful for this project's user base?
- Should the system evolve toward a report-oriented pipeline, or is the current analysis-tool model the intended end state?

