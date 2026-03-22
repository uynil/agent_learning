---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: test whether the reflection loop helps on a broader query
---

# Deep Research Mode

## Research Loop Overview

This project implements deep research as a fixed sectioned loop rather than as a broad free-form investigation mode.

The sequence is:

1. generate a report structure
2. research each paragraph
3. summarize each round
4. reflect and search again
5. revise the paragraph summary
6. format the final report

## Question Rewrite Strategy

- The user query is not rewritten into a new standalone research question by a dedicated planner.
- Instead, `ReportStructureNode` turns the query into a small set of report sections.
- That sectioning step acts as a coarse decomposition mechanism.
- `ReflectionNode` performs a narrower local rewrite for the current paragraph by using the latest paragraph state.

## Decomposition Strategy

- Decomposition happens at the report-section level.
- The report structure prompt caps the outline at five paragraphs.
- Each paragraph becomes one research thread.
- This gives the project a multi-round shape, but the decomposition is section-oriented rather than question-tree-oriented.

## Evidence Accumulation

- Each search pass contributes new search results to `search_history`.
- Each summary pass condenses those results into `latest_summary`.
- Reflection uses the latest summary as the next context seed.
- The final report is built from the latest summary of every paragraph.

## Reflection and Revision

- `ReflectionNode` asks what is still missing from the current paragraph state.
- It generates a new search query based on that gap analysis.
- `ReflectionSummaryNode` merges the new evidence into the paragraph summary.
- `reflection_iteration` records how many refinement passes happened.

## Stop Conditions

- The per-paragraph loop stops after `Config.max_reflections`.
- The whole process stops after all paragraphs are marked complete.
- There is no adaptive early-stop logic beyond those loop bounds.

## Resumability and Context Handoff

- The saved JSON state makes interruption recovery possible.
- A resumed session can restore the report structure, paragraph summaries, and search history.
- However, resume is manual, not a dedicated deep-research entry point.
- The project does not appear to store a dedicated round-summary artifact separate from the full state file.

## Reusable Patterns

- per-section deep research loops
- reflection-driven search query revision
- summary-as-context for subsequent passes
- bounded iteration for stopping
- persisted state for manual resumption

## Open Questions

- Would a real query-rewrite planner produce better research sections than the current report-structure step?
- Is the current reflection loop enough for deep research, or does it need a broader synthesis phase?
- Should resumability have a dedicated user-facing workflow?
