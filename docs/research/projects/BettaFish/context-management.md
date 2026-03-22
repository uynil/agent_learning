---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: trace per-node payload builders if context-window usage needs a tighter estimate
---

# Context Management

## Observed Facts

- Query, Insight, and Media each keep a `State` object with `paragraphs`, `Research`, and `Search` records. The common fields are `search_history`, `latest_summary`, `reflection_iteration`, and `is_completed`.
- `MediaEngine/state/state.py` extends the search record with `paragraph_title`, `search_tool`, and `has_result`, which makes it easier to preserve provenance when a search returns nothing.
- `ReportEngine/state/state.py` stores only the minimal task state needed for the final composition pass: query, status, selected template, HTML content, and metadata.
- `ReportEngine/core/chapter_storage.py` writes a per-report run directory, `manifest.json`, `stream.raw`, and final `chapter.json` files, so chapter context survives beyond the model call.
- `ReportEngine/core/stitcher.py` uses the manifest metadata to inject anchors, sort chapters, and preserve `themeTokens` and `assets`.
- `ForumEngine` uses `forum.log` as shared context. The monitor clears it at the start of a session, writes host and agent speech, and filters out system messages.
- `TemplateSelectionNode` truncates long reports to 1000 characters and forum logs to 800 characters before sending them to the model.
- `ChapterGenerationNode` keeps the last 8000 characters of a failed chapter output when building a recovery payload.
- `app.py` treats the forum log as a live stream source and forwards it into SocketIO and the report interface.

## Inferences

- The system uses a mix of in-memory state and durable artifacts, but the durable artifacts are the real memory layer. This makes multi-step generation debuggable after the fact.
- Context is progressively compressed as the pipeline moves forward: raw crawl data becomes paragraph state, paragraph state becomes report sections, and report sections become chapter JSON.
- The truncation strategy suggests the system values breadth and recoverability over perfect retention of every upstream detail.

## Open Questions

- How often the 1000-character and 800-character truncation limits remove important evidence from template selection.
- Whether the research engines keep enough per-paragraph provenance to reconstruct search decisions later.
- How much of the final report context depends on forum logs versus the three engine summaries.

## Next Action

If context handling becomes a focus, inspect the individual node builders that assemble search and summary payloads, then compare them against the state serialization paths.
