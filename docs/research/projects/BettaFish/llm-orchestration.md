---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare node-level search and summary prompts if fine-grained model routing matters
---

# LLM Orchestration

## Observed Facts

- `QueryEngine/llms/base.py`, `InsightEngine/llms/base.py`, and `MediaEngine/llms/base.py` all wrap OpenAI-compatible chat completions with retry support and streaming helpers.
- Query, Insight, and Media each prefix the current time into the user prompt before making a request. `ReportEngine/llms/base.py` does not add that prefix.
- `ReportEngine` keeps one primary LLM client and builds a fallback list for chapter repair using the forum, insight, and media models when available.
- `QueryEngine` and `InsightEngine` use the same node sequence: report structure, first search, reflection, first summary, reflection summary, and final formatting.
- `MediaEngine` follows the same orchestration pattern, but its tool layer is multimodal and its prompt contracts are narrower around web/image/structured-data search.
- `ReportEngine` uses a different orchestration chain: template selection, document layout, word budgeting, chapter generation, IR stitching, and rendering.
- `ChapterGenerationNode` has multiple repair paths: local JSON repair, cross-engine rescue, structural repair, and content-density checks that can force retries.
- `ForumEngine/llm_host.py` uses `OpenAI` directly plus graceful retry, and injects a current-time prefix before asking the host model to respond.

## Inferences

- The project uses orchestration by specialization rather than by a single general-purpose agent. Each engine has one narrow job and one prompt family.
- Time-prefixing in the research engines is a small but deliberate recency cue, especially useful for search-heavy tasks where freshness matters.
- Report generation is the most defensive orchestration path. It treats malformed chapter JSON as a normal failure mode and tries several recovery routes before giving up.

## Open Questions

- Whether the retry behavior is tuned differently per engine in `retry_helper.py`, or whether the same backoff logic is shared everywhere.
- How often the cross-engine chapter rescue path succeeds versus simply falling back to a placeholder chapter.
- Whether the forum host materially changes downstream research quality or primarily improves traceability.

## Next Action

If orchestration needs a deeper audit, inspect the individual `search_node.py` and `summary_node.py` files to compare how each engine chooses tools and summarizes evidence.
