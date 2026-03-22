---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect source-state persistence and recovery paths
---

# Architecture

## Architecture Summary

`gpt-researcher` is a web-first research pipeline built around FastAPI, WebSockets, search, scraping, chunked summarization, and final report synthesis.

## Module Breakdown

- `examples/gpt-researcher/main.py` exposes the web app and WebSocket entry point.
- `examples/gpt-researcher/agent/run.py` coordinates a research run.
- `examples/gpt-researcher/agent/research_agent.py` contains the research workflow itself.
- `examples/gpt-researcher/agent/prompts.py` contains the prompt builders.
- `examples/gpt-researcher/agent/llm_utils.py` wraps model calls and agent selection.
- `examples/gpt-researcher/actions/web_search.py` handles web search.
- `examples/gpt-researcher/actions/web_scrape.py` handles browser-based page extraction and page summarization.
- `examples/gpt-researcher/processing/text.py` handles chunking, summarization, and markdown/pdf output.
- `examples/gpt-researcher/config/config.py` manages runtime configuration.

## Execution Flow

1. A user sends a task through the WebSocket.
2. `main.py` optionally chooses a domain agent for the task.
3. `run.py` creates `ResearchAgent`.
4. `ResearchAgent.conduct_research()` generates search queries and collects summaries.
5. `ResearchAgent.write_report()` generates the final report.
6. The report is written to Markdown and converted to PDF.

## State and Data Flow

- `question` is the primary input.
- `agent_role_prompt` controls the persona used in LLM calls.
- `visited_urls` avoids repeated scraping.
- `research_summary` accumulates the textual evidence used by later prompts.
- `dir_path` stores per-task output artifacts under a hash of the question.

## Extensibility

- New personas can be added through `generate_agent_role_prompt()`.
- New report styles can be added through `get_report_by_type()`.
- Search and scraping backends can be swapped independently of the prompt layer.

## Tradeoffs

- The architecture is easy to understand but mostly linear.
- It favors fast implementation over deep state management.
- The summary accumulation strategy is simple and practical, but it can become brittle if the source summaries are noisy.

## Open Questions

- Whether the separate `permchain_example` is an alternative architecture or a prototype of a more agentic design.
- Whether report quality depends more on search coverage or on the final synthesis prompt.
- Whether the current output persistence model is enough for interrupted or resumed work.
