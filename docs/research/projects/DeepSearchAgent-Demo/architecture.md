---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: validate the module boundaries against a live run
---

# Architecture

## Summary

This project is organized as a single-agent, node-based workflow. `DeepSearchAgent` orchestrates a fixed pipeline, while specialized node classes handle structure generation, search query generation, summary updates, reflection, and final formatting.

## Module Breakdown

- `src/agent.py`
  Main orchestrator. It creates the LLM client, initializes nodes, runs the research loop, saves state, and emits the final report.
- `src/nodes/`
  Stage-specific processing units.
  - `ReportStructureNode`
  - `FirstSearchNode`
  - `FirstSummaryNode`
  - `ReflectionNode`
  - `ReflectionSummaryNode`
  - `ReportFormattingNode`
- `src/prompts/prompts.py`
  Houses all system prompts and JSON schema definitions.
- `src/state/state.py`
  Defines the persistent state model for the report and paragraph research progress.
- `src/llms/`
  Thin provider adapters for DeepSeek and OpenAI.
- `src/tools/search.py`
  Tavily search wrapper.
- `src/utils/text_processing.py`
  JSON cleanup, markdown cleanup, and result truncation helpers.

## Execution Flow

1. `DeepSearchAgent.research()` creates a report structure for the query.
2. The agent iterates through each paragraph produced by that structure.
3. For each paragraph, the agent generates a search query, runs Tavily search, and produces an initial summary.
4. The agent then enters a bounded reflection loop to find missing angles and deepen the paragraph.
5. After all paragraphs are finished, the agent formats the final Markdown report.
6. The report and state can be saved to disk.

## State and Data Flow

- The top-level `State` keeps the query, report title, paragraph list, final report, and timestamps.
- Each `Paragraph` has a `Research` object with search history, latest summary, reflection count, and completion flag.
- Each `Search` record stores the query and raw search result metadata.
- `latest_summary` is the main working-memory slot used by later rounds.
- `search_history` is the archival trail used for traceability and later review.
- The saved JSON state makes the workflow resumable, at least manually.

## Extensibility

- New LLM providers can be added by implementing `BaseLLM`.
- New node types can slot into the current pipeline if they follow the existing input/output conventions.
- Search can be swapped or augmented by replacing `tavily_search`.
- Prompt behavior can be changed without touching the orchestration shape, because prompts are centralized in `src/prompts/prompts.py`.

## Tradeoffs

- The design is simple and easy to trace, which is a strength for research work.
- The same simplicity also makes the pipeline rigid.
- There is no explicit planner/executor/reviewer abstraction outside the node names.
- There is no parallelism across sections.
- Reliability depends heavily on the JSON contract and post-processing helpers.
- The architecture is good for controlled research reports, but less suitable for open-ended autonomous exploration.

## Open Questions

- Should the project make the section-level loop more explicit as a reusable plan/execution abstraction?
- Would a more formal retry layer improve robustness for JSON failures or weak search results?
- Should paragraph generation be capped by the config value rather than only by prompt wording?
