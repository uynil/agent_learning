---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: install dependencies and API keys, then live-run a sample task before deepening conclusions
---

# Notes

## Session Notes

- First-pass reading suggests `gpt-researcher` is a practical, stage-driven research agent rather than a deeply iterative reasoning system.
- The project is strongest at gathering and compressing web evidence quickly.
- Prompt structure is clear but simple.
- A first runtime validation attempt failed before task execution because the local Python environment is missing the `openai` package and `OPENAI_API_KEY` is unset.

## What I Understood

- The project uses separate prompts for classification, query generation, summarization, and final report generation.
- The main research memory is the accumulated summary text rather than a richer graph of claims or evidence.
- Parallel scraping is a major part of the design.

## What I Changed My Mind About

- I initially expected more explicit deep research behavior, but the main path is more of a breadth-first research pipeline.
- The project does not appear to rely on heavy prompt layering; instead, it relies on clean orchestration and repeated summarization.

## Worth Revisiting

- Whether `generate_concepts_prompt()` is actually usable as written in live runs.
- Whether a live query reveals more about report quality than the static code path.
- Whether the `permchain_example` should be compared later as a more multi-actor variant.

## Linked Comparisons

- `docs/research/comparisons/prompt-structure-patterns.md`
- `docs/research/comparisons/prompt-function-mapping-patterns.md`
- `docs/research/comparisons/llm-scheduling-patterns.md`

## Blocked or Low-Confidence Record

- Status: blocked for live validation
- Files Checked: `README.md`, `agent/prompts.py`, `agent/research_agent.py`, `agent/llm_utils.py`, `agent/run.py`, `main.py`, `config/config.py`, `actions/web_search.py`, `actions/web_scrape.py`, `processing/text.py`
- Blocker: importing `config.config` already fails with `ModuleNotFoundError: No module named 'openai'`; current shell also has `OPENAI_API_KEY` unset
- Working Hypothesis: the app is a planner/executor pipeline with simple but effective prompt staging
- Resume From: install `requirements.txt`, export `OPENAI_API_KEY`, then run one real task and compare the observed call sequence to the static read

## Next Research Plan

1. Install runtime dependencies from `requirements.txt`.
2. Set `OPENAI_API_KEY`.
3. Run a live sample task.
4. Verify query generation and report synthesis behavior.
5. Reassess whether a deeper `architecture` or `deep-research-mode` comparison is warranted.
