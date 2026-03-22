---
project: MultiAgent-Data-Analyst
source_path: examples/MultiAgent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: verify the canonical stack and repair the broken resume path before claiming end-to-end maturity
---

# Notes

- The repo is more useful as a learning case than as a polished production architecture.
- The observed architecture here is `src/` + `streamlit_app/` + `mcp/`, not a separate `app/`-based orchestrator stack.
- This repo is more like a staged analytics pipeline with A2A messaging and notebook synthesis.
- There are at least two memory models in play, and neither is fully cleanly wired across every caller.
- `ModelAgent` looks unfinished in the current source tree, which weakens the “multi-agent AutoML” story.
- The prompt layer is intentionally local and task-specific, not centralized.
- The strongest prompt technique in the repo is post-hoc narrative synthesis from structured artifacts.
- Because the neighboring repo has an almost identical name, this project is easy to mentally conflate with the more LLM-heavy `Multi-Agent-Data-Analyst`.

## What I Would Verify Next

1. Decide which runtime path is canonical.
2. Fix the `MemoryTools.load()` call pattern.
3. Confirm whether `ModelAgent._train_and_evaluate()` exists elsewhere or needs to be implemented.
4. Run one real dataset through the selected path and record the outputs.

## Comparison Hint

Compared with `Multi-Agent-Data-Analyst`, this project feels less like a unified LLM orchestrator and more like a broader analytics workflow demo with thinner LLM involvement.
