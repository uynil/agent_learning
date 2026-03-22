---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: install runtime dependencies, set API keys, and retry a sample task
---

# gpt-researcher

This file is the canonical entry page for future research notes on `examples/gpt-researcher`.

## Project Summary

`gpt-researcher` is a web-research agent that turns a user task into a set of research queries, crawls the web in parallel, summarizes sources chunk by chunk, and then generates a long-form report in a selected role.

## Quick Classification

- Style: planner / executor with post-hoc report synthesis
- Research shape: search -> scrape -> summarize -> report
- Prompt role: prompts define both task routing and report generation

## Key Files

- `examples/gpt-researcher/main.py`
- `examples/gpt-researcher/agent/research_agent.py`
- `examples/gpt-researcher/agent/prompts.py`
- `examples/gpt-researcher/agent/llm_utils.py`
- `examples/gpt-researcher/processing/text.py`
- `examples/gpt-researcher/actions/web_search.py`
- `examples/gpt-researcher/actions/web_scrape.py`
- `examples/gpt-researcher/config/config.py`

## Current Status

The main research path is easy to trace and heavily prompt-driven. The project already has a clear separation between query generation, source gathering, summarization, and final report writing.

## Next Action

Install missing runtime dependencies, set `OPENAI_API_KEY`, then retry a sample task before deepening runtime conclusions.

## Observed Facts

- The app is exposed through FastAPI and a WebSocket endpoint in `examples/gpt-researcher/main.py`.
- The main execution path instantiates `ResearchAgent` and runs `conduct_research()` before `write_report()`.
- The agent uses a small set of prompt builders in `examples/gpt-researcher/agent/prompts.py`.
- Search uses DuckDuckGo via `examples/gpt-researcher/actions/web_search.py`.
- Scraping uses Selenium plus BeautifulSoup in `examples/gpt-researcher/actions/web_scrape.py`.
- Summaries are created chunk-by-chunk in `examples/gpt-researcher/processing/text.py`.
- Configuration is environment-driven in `examples/gpt-researcher/config/config.py`.

## Inferences

- The project is optimized for breadth-first web research rather than long-lived agent memory.
- The core reliability strategy is parallel source gathering plus repeated summarization, not stateful planning memory.
- Prompt design is intentionally lightweight and mostly template-driven.

## Open Questions

- How much the current report quality depends on the exact `gpt-4` / `gpt-3.5-turbo-16k` split.
- Whether the concept-generation path is fully functional in practice.
- Whether the separate `permchain_example` represents a future direction or just a side experiment.
- After dependencies and API keys are configured, does the observed runtime still match the current planner/executor reading?

## Project Notes

- Additional deep-dive notes can be added later if the project proves worth comparing against other research agents.
