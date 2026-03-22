---
project: BettaFish
source_path: examples/BettaFish
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect node prompts if you want a prompt-by-prompt delta table
---

# Prompts Analysis

## Observed Facts

- `QueryEngine`, `InsightEngine`, and `MediaEngine` all use the same prompt scaffold: report structure, first search, first summary, reflection, reflection summary, and report formatting.
- Query prompts focus on web-news search. Insight prompts focus on local social data, sentiment analysis, and platform-specific behavior. Media prompts focus on multimodal search and structured-data retrieval.
- The reflection prompts are intentionally more conversational than the first-pass search prompts. They push the model to think about missing evidence and to choose more natural search terms.
- `ReportEngine/prompts/prompts.py` is much stricter than the research-engine prompts. It embeds JSON schemas, block-type constraints, and explicit rules for `engineQuote`, `swotTable`, and `pestTable`.
- `ReportEngine` asks for Chinese numbering, pure JSON responses, and explicit section-level metadata such as `tocPlan`, `hero`, and `themeTokens`.
- The forum host prompt is narrative and facilitative rather than schema-heavy. It asks for timeline, comparison, deeper analysis, and discussion questions.
- `ReportEngine` prompt rules are reinforced by tests in `tests/test_report_engine_sanitization.py`, which verify `engineQuote` validation and block sanitization.

## Inferences

- There are two prompt styles in the project: extractive/structured prompts for machine-to-machine handoff, and deliberative prompts for planning and moderation.
- The prompt system is intentionally redundant. It relies on both prompt instructions and code-level validation so that the final output remains stable even when the model drifts.
- The research engines appear optimized for iterative evidence gathering, while the report engine is optimized for contract compliance.

## Open Questions

- Whether the prompt length in `ReportEngine` or `ForumEngine` ever causes context pressure on smaller models.
- Whether there is a separate prompt library outside the codebase that mirrors these contracts for human editing.
- How much prompt tuning differs between Query, Insight, and Media beyond the shared scaffold.

## Next Action

If we want a more detailed prompt map, the next useful step is a per-node comparison of the `search_node`, `summary_node`, and `formatting_node` files across the three research engines.
