---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: validate the call sequence with one real end-to-end research run
---

# LLM Orchestration

## LLM Call Map

| Stage | Node | Responsibility |
| --- | --- | --- |
| 1 | `ReportStructureNode` | generate the report outline |
| 2 | `FirstSearchNode` | create the first search query for a paragraph |
| 3 | `FirstSummaryNode` | synthesize the first paragraph draft from search results |
| 4 | `ReflectionNode` | derive a follow-up search query from the latest paragraph state |
| 5 | `ReflectionSummaryNode` | revise the paragraph using new evidence |
| 6 | `ReportFormattingNode` | format the final report as Markdown |

## Scheduling Pattern

- The workflow is sequential and section-based.
- The agent processes one paragraph at a time.
- Each paragraph gets one initial search pass and up to `Config.max_reflections` reflection passes.
- The final report is only generated after all paragraphs are marked complete.
- There is no parallel section processing.

## Structured Output Strategy

- Every stage uses JSON schemas embedded in the prompt.
- The code expects JSON objects or arrays with stage-specific fields.
- The nodes clean fence markers and reasoning text before parsing.
- `extract_clean_response()` is used when direct JSON parsing fails.
- The final stage expects Markdown, but still has a manual formatter fallback.

## Context Handoff

- `State.query` is the top-level user request.
- `State.report_title` is set by the structure stage.
- `Paragraph.content` holds the planned intent for that section.
- `Paragraph.research.latest_summary` is the main handoff object between search and reflection rounds.
- `Paragraph.research.search_history` preserves the evidence trail.
- `ReflectionNode` receives the latest summary and turns it into a follow-up query.
- `ReflectionSummaryNode` writes the revised section content back into state.
- `State.save_to_file()` and `State.load_from_file()` make manual continuation possible.

## Reliability Mechanisms

- JSON-only instructions reduce free-form drift.
- `remove_reasoning_from_output()` strips common reasoning wrappers.
- `clean_json_tags()` removes fenced code blocks.
- `extract_clean_response()` tries multiple JSON recovery strategies.
- `FirstSearchNode` and `ReflectionNode` fall back to default queries if parsing fails.
- `ReportFormattingNode` falls back to a manual formatter if the LLM formatting step fails.
- Search is bounded by `max_search_results` and `search_timeout`.
- The reflection loop is bounded by `max_reflections`.

## Strengths

- The call graph is easy to understand and trace.
- Every stage has a clear responsibility.
- The workflow is resilient enough to keep going even if one formatting step fails.
- The saved state model makes the process resumable.
- The architecture is flexible enough to switch between DeepSeek and OpenAI.

## Limitations

- The orchestration is rigid and highly sequential.
- There is no explicit planner/reviewer separation beyond the stage names.
- There is no retry policy beyond fallback defaults and manual formatting.
- The workflow depends on the quality of the generated search queries.
- Reflection is paragraph-local rather than global across the whole report.
- There is no explicit recovery UX for partially completed sessions.

## Questions for Further Research

- Does the reflection loop actually improve section quality in practice?
- Would adding a global reviewer stage improve coverage or just add cost?
- Should section generation and section execution be separated more explicitly?
- Would a formal resume flow make the state model more useful?
