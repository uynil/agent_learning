---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect node-level search/summary files if you want a deeper evidence ledger
---

# Notes

## Observed Facts

- Reviewed source files: `app.py`, `README.md`, `README-EN.md`, `MindSpider/main.py`, `ForumEngine/monitor.py`, `ForumEngine/llm_host.py`, `QueryEngine/agent.py`, `InsightEngine/agent.py`, `MediaEngine/agent.py`, `ReportEngine/agent.py`, `ReportEngine/flask_interface.py`, `ReportEngine/prompts/prompts.py`, `QueryEngine/prompts/prompts.py`, `InsightEngine/prompts/prompts.py`, `MediaEngine/prompts/prompts.py`, `ReportEngine/state/state.py`, `QueryEngine/state/state.py`, `InsightEngine/state/state.py`, `MediaEngine/state/state.py`, `ReportEngine/core/*`, `ReportEngine/ir/*`, `ReportEngine/renderers/html_renderer.py`, `tests/test_monitor.py`, and `tests/test_report_engine_sanitization.py`.
- The code is already organized around a strong split between research and composition.
- The report engine has the most explicit safety rails: JSON repair, structural validation, content-density checks, and renderer-side chart validation.
- The forum monitor is a real integration point, not a decorative log viewer.

## Inferences

- BettaFish is closer to a pipeline system than a single chatbot. Each stage hands off a different kind of artifact to the next stage.
- The most reusable idea in the codebase is the contract boundary, not any one model call.
- The most fragile area is likely the boundary between free-form model output and structured IR, which is why the repair code is so prominent.

## Open Questions

- Which node files are still worth reading if we want a second-pass comparison of prompt and tool selection behavior.
- Whether the single-engine Streamlit wrappers expose custom controls or simply proxy the backend agents.
- How representative the bundled tests are of the project’s actual failure modes.

## Next Action

If you continue the research, start with the node-level search and summary files for the three research engines, then inspect the HTML renderer’s widget/chart behavior only if the report presentation layer becomes relevant.
