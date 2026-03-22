---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: install dependencies and API keys, then retry the first live research run
---

# Notes

## Session Notes

This first pass shows a very clear staged research pipeline: outline, search, summarize, reflect, revise, format.

A first live-run attempt did not reach the agent loop. It failed earlier because the local environment is not ready yet.

## What I Understood

- The project is designed for controlled deep research, not open-ended chat.
- The summary state is the key handoff point between rounds.
- Prompt quality depends heavily on JSON schema alignment and cleanup helpers.

## What I Changed My Mind About

- I initially treated the prompt layer as mostly a reliability problem.
- After reading the code, it is also a structure problem and a mechanism problem: the prompts are doing deliberate work, not just constraining output.

## Worth Revisiting

- Whether `max_paragraphs` should be enforced in code.
- Whether the reflection loop adds enough new evidence on broad topics.
- Whether there should be a first-class resume workflow.

## Linked Comparisons

- `docs/research/comparisons/prompt-structure-patterns.md`
- `docs/research/comparisons/prompt-function-mapping-patterns.md`
- `docs/research/comparisons/context-management-patterns.md`
- `docs/research/comparisons/deep-research-patterns.md`

## Blocked or Low-Confidence Record

- Status: blocked for live validation
- Files Checked: `src/agent.py`, `src/prompts/prompts.py`, `src/state/state.py`, `src/nodes/*`, `src/llms/*`, `src/tools/search.py`, `src/utils/*`
- Blocker: `python3 examples/basic_usage.py` fails with `ModuleNotFoundError: No module named 'openai'`; current shell also has `OPENAI_API_KEY`, `DEEPSEEK_API_KEY`, and `TAVILY_API_KEY` unset
- Working Hypothesis: this is a sectioned deep-research pipeline with strong JSON contracts
- Resume From: install `requirements.txt`, export the required API keys, rerun `examples/basic_usage.py`, then compare one real run against these assumptions

## Next Research Plan

1. Install runtime dependencies from `requirements.txt`.
2. Set `OPENAI_API_KEY` or `DEEPSEEK_API_KEY`, plus `TAVILY_API_KEY`.
3. Run a live query.
4. Compare the actual report structure, search queries, and reflection behavior against the current static analysis.
