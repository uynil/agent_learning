---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare function-to-mechanism mapping against the orchestration layer on a real run
---

# Prompts Analysis

## Prompt Inventory

- `SYSTEM_PROMPT_REPORT_STRUCTURE`
  Generates the report outline and section list.
- `SYSTEM_PROMPT_FIRST_SEARCH`
  Generates the first search query for a paragraph.
- `SYSTEM_PROMPT_FIRST_SUMMARY`
  Turns the first search results into a paragraph summary.
- `SYSTEM_PROMPT_REFLECTION`
  Generates a follow-up search query from the latest paragraph state.
- `SYSTEM_PROMPT_REFLECTION_SUMMARY`
  Revises the paragraph using the new search results.
- `SYSTEM_PROMPT_REPORT_FORMATTING`
  Formats the final report into Markdown.

## Prompt Function Map

| Prompt | Primary Function | Key Output |
| --- | --- | --- |
| `SYSTEM_PROMPT_REPORT_STRUCTURE` | plan report shape | list of paragraph skeletons |
| `SYSTEM_PROMPT_FIRST_SEARCH` | decide first search query | `search_query`, `reasoning` |
| `SYSTEM_PROMPT_FIRST_SUMMARY` | synthesize paragraph draft | `paragraph_latest_state` |
| `SYSTEM_PROMPT_REFLECTION` | discover missing angles | `search_query`, `reasoning` |
| `SYSTEM_PROMPT_REFLECTION_SUMMARY` | revise paragraph with new evidence | `updated_paragraph_latest_state` |
| `SYSTEM_PROMPT_REPORT_FORMATTING` | produce readable final report | Markdown report |

## Prompt Structure

Each prompt is a single system prompt plus a single user payload passed through `BaseLLM.invoke(system_prompt, user_prompt)`.

The prompts are strongly structured:

- they define an input schema in the prompt body
- they define an output schema in the prompt body
- they tell the model to return JSON only
- they usually add a Chinese-language output constraint
- they keep the task narrow and stage-specific

## Core Prompt Elements

- Role framing: every prompt casts the model as a “deep research assistant”.
- Task framing: each prompt asks for one narrow step, not the whole report.
- Input schema: the model sees the exact fields it should consume.
- Output schema: the model is pushed toward a machine-readable shape.
- Tool instruction: the search and reflection prompts mention a search tool.
- Output constraint: “只返回JSON对象，不要有解释或额外文本”.
- Stage isolation: each prompt is tied to one node in the workflow.
- Reflection hook: the reflection prompts use the latest paragraph state as a prompt input.

## Function-to-Mechanism Mapping

| Function | Mechanism | Why It Works |
| --- | --- | --- |
| plan report structure | limit to 5 paragraphs and require title/content pairs | it constrains decomposition early |
| generate search query | schema + one-task prompt + reasoning field | it keeps query generation explicit |
| summarize findings | search results passed in truncated form | it grounds the summary in recent evidence |
| reflect on gaps | latest summary becomes the input | it turns the previous round into context for the next round |
| update paragraph state | dedicated summary-update prompt | it separates synthesis from orchestration |
| format final report | final formatting prompt plus manual fallback | it keeps the output readable even if formatting fails |

## Prompt Organization Pattern

- role-based at the broadest level
- stage-based at the implementation level
- heavily schema-driven
- not dynamically assembled from many smaller prompt fragments
- tightly coupled to node classes

## Prompt Composition Strategy

- system prompt for each stage
- JSON schema embedded directly into the prompt text
- one user payload per invocation
- post-processing in code for cleanup and repair

## Prompt-to-Code Coupling

- `ReportStructureNode` imports `SYSTEM_PROMPT_REPORT_STRUCTURE`
- `FirstSearchNode` and `ReflectionNode` import their search prompts
- `FirstSummaryNode` and `ReflectionSummaryNode` import their summary prompts
- `ReportFormattingNode` imports the final formatting prompt
- `text_processing.py` repairs malformed output after prompt execution
- `BaseLLM.invoke()` is the common interface, so the prompt layer and orchestration layer are closely linked

## Constraint and Reliability Strategy

- JSON schema is written into the prompt
- output is expected to be JSON only
- the code strips reasoning markers and code fences before parsing
- malformed JSON is recovered with fallback extraction
- broken search or formatting can fall back to default queries or manual report formatting

## Likely Failure Modes

- the model may return valid language but invalid JSON
- `remove_reasoning_from_output()` may over-strip text in edge cases
- summaries may become repetitive if reflection queries do not add new evidence
- prompt constraints may drift from actual code behavior if config values change
- `max_paragraphs` is present in config but not clearly enforced by runtime logic

## Reusable Prompt Patterns

- one prompt per stage
- explicit input/output schema
- narrow task scope
- summary-as-context for the next round
- final formatting as a separate post-processing step
- manual fallback outside the LLM path
