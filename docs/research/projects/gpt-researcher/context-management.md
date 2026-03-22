---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect whether recovery/resume is first-class
---

# Context Management

## Context Sources

- The original user task.
- The selected agent role prompt.
- Search queries generated from the task.
- Per-page summaries generated from scraped content.
- The accumulated `research_summary`.
- The visited URL set.

## Memory and State Layers

- `visited_urls` is the clearest persistent state object in the main path.
- `research_summary` acts as the working memory for later report synthesis.
- Output files under `./outputs/<hash>` act as a crude session record.

## Retrieval and Selection Strategy

- Search queries are generated first, then results are filtered by browser scraping.
- Duplicate URLs are dropped through the visited set.
- Only the most relevant text fragments survive into summary form.

## Context Packing Strategy

- Large pages are split into chunks before summarization.
- Each chunk is summarized independently with a short prompt.
- Chunk summaries are then summarized again into one final page summary.
- The final report prompt receives the accumulated `research_summary` as context.

## Context Update Rules

- A scraped page turns into a summary string.
- A summary string is appended to `research_summary`.
- Revisited URLs are ignored.
- The per-question output directory stores intermediate text files for later reuse.

## Failure Modes

- Noise can accumulate if early summaries are weak.
- The summary stack can flatten nuance from the original sources.
- There is no obvious first-class memory retrieval layer beyond the saved text files.
- Resumption looks file-based rather than stateful.

## Reusable Patterns

- Maintain a compact working summary instead of carrying full raw pages forward.
- Keep a deduplicated set of visited sources.
- Use chunk-level summarization for long pages.
- Persist intermediate artifacts to disk so a later pass can continue from files instead of memory.

## Open Questions

- How well the saved `.txt` artifacts support resumed work in practice.
- Whether the project needs stronger source ranking or memory retrieval.
- Whether the concept-generation path is intended as a second-stage memory refinement or just an auxiliary feature.
