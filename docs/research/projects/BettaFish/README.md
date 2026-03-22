---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect remaining node files if deeper prompt-level detail is needed
---

# BettaFish

BettaFish is a multi-agent public-opinion analysis system that combines upstream crawling, three parallel analysis engines, a forum-style collaboration layer, and a final report composer. The codebase is already split into clear subsystems, so this research set focuses on how those subsystems connect rather than on any one model or UI.

## Observed Facts

- `examples/BettaFish/app.py` boots `MindSpider`, the three Streamlit-based single-engine apps, `ForumEngine`, and `ReportEngine` in one process.
- `QueryEngine`, `InsightEngine`, and `MediaEngine` all follow the same four-stage loop: structure the report, search, summarize, reflect, then format the result.
- `ForumEngine` listens to log output, extracts summary-node speech, and turns it into moderator-style host messages.
- `ReportEngine` turns the upstream engine outputs into a template-based document IR, then renders HTML and PDF.

## Inferences

- The project is designed around staged synthesis: gather data first, let each engine specialize, then fuse everything into a single report pass.
- The forum layer is not just logging; it acts like a control loop that encourages the three research engines to refine their direction.
- BettaFish favors file-backed artifacts and streamable logs over ephemeral in-memory state, which makes the workflow easier to inspect and recover.

## Open Questions

- How much quality difference exists between the three research engines once their node-level prompts and tool wrappers are compared side by side?
- How often does the forum loop actually change the search direction in practice rather than just narrating it?
- Which render-time behaviors depend on template content versus IR validation versus chart repair?

## Next Action

Review the remaining node implementations only if a second pass needs finer-grained evidence about search-node behavior, chapter repair, or rendering edge cases.
