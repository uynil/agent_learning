---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect context management and retry behavior
---

# LLM Orchestration

## LLM Call Map

1. `choose_agent()` in `examples/gpt-researcher/agent/llm_utils.py` classifies the task into a domain agent.
2. `ResearchAgent.create_search_queries()` asks the model for three search queries.
3. `processing.text.summarize_text()` calls the model once per text chunk, then once more on the combined summaries.
4. `ResearchAgent.write_report()` calls the model to synthesize the final report.
5. `ResearchAgent.write_lessons()` can call the model again for each concept lesson.

## Scheduling Pattern

- The system is planner / executor shaped.
- The planner side generates search questions and optionally picks a domain persona.
- The executor side performs web search, web scraping, and chunked summarization.
- The final synthesis step aggregates all collected summaries into a report.
- Source gathering is parallelized with `asyncio.gather()` in `ResearchAgent.async_search()`.

## Structured Output Strategy

- Search queries are expected as a JSON list of strings.
- Concepts are expected as a JSON list of strings.
- Final reports are Markdown documents.
- PDF output is generated after Markdown is written to disk.

## Context Handoff

- The main handoff unit is `self.research_summary` on `ResearchAgent`.
- `research_summary` is built from saved `.txt` files and appended search results.
- Each web page summary becomes part of the next-stage synthesis context.
- The code does not appear to maintain a richer long-term memory structure beyond the accumulated summary and visited URL set.

## Reliability Mechanisms

- Multiple sources are scraped per query.
- Search and scrape happen in parallel.
- Each page is summarized in chunks to avoid feeding huge documents into one call.
- `visited_urls` prevents reprocessing the same source URL.
- The model is asked to cite facts and numbers when available.

## Strengths

- Clear stage separation.
- Parallel source collection improves throughput.
- Chunked summarization is a practical way to control token pressure.
- Final synthesis is grounded in accumulated web-derived evidence rather than only the user question.

## Limitations

- Reliability depends heavily on the quality of web search results.
- The orchestration is breadth-oriented, not deeply reflective.
- The system does not show a first-class recovery flow for interrupted research sessions.
- The current orchestration is more about aggregation than iterative reasoning.

## Questions for Further Research

- How often does the query-generation step produce useful coverage versus redundant queries?
- How much quality depends on the selected domain agent persona?
- Whether the concept / lesson path is actually used in real runs.
- Whether the current orchestration is resilient enough for repeated, long-running research tasks.
