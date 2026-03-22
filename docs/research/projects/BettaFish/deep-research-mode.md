---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: measure how many reflection cycles the research engines usually need on a real topic
---

# Deep Research Mode

## Observed Facts

- `QueryEngine/agent.py`, `InsightEngine/agent.py`, and `MediaEngine/agent.py` all implement the same deep-research loop: generate a report structure, process each paragraph, search, summarize, reflect, refine, and then format.
- `QueryEngine` uses internet search tools; `InsightEngine` uses a local social database plus keyword optimization and sentiment analysis; `MediaEngine` uses multimodal search and structured-data search.
- The forum monitor watches summary-node output, not search-node output. That keeps the discussion loop focused on synthesized findings instead of raw tool chatter.
- `ForumEngine` generates a host speech after collecting several agent speeches. The threshold is five speeches, based on the monitor state.
- `ForumEngine/llm_host.py` asks the host model to produce a structured answer with event timeline, viewpoint comparison, deeper analysis, and future discussion questions.
- `MindSpider` can run a two-step upstream workflow: broad topic extraction first, then deep sentiment crawling.
- The report engine’s chapter generator can recover from malformed JSON by using local repair, cross-engine rescue, or placeholder chapters, which keeps deep generation moving even when the model stumbles.

## Inferences

- BettaFish’s “deep research” is not a single agent call. It is a recursive loop where each engine revises its own search plan after reading its latest summary and the forum discussion.
- The forum layer acts like a meta-researcher: it aggregates what the three engines have said and pushes them toward underexplored angles.
- The report engine is part of the deep-research mode, not just a renderer. Its chapter-level retries and repair paths are what let the system turn messy evidence into a polished report.

## Open Questions

- How many reflection loops are usually enough before each engine stabilizes on a useful answer.
- Whether forum host messages actually change the next search query or mostly improve readability and traceability.
- How often `MindSpider` is run as a separate preparation step versus being triggered as part of the same operator workflow.

## Next Action

Run a real topic through the workflow and capture one pass of the forum log plus the final report artifacts. That would show whether the reflection loop, host moderation, and chapter repair are all pulling their weight or just adding ceremony.
