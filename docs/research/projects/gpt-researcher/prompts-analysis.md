---
project: gpt-researcher
source_path: examples/gpt-researcher
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect orchestration and context flow
---

# Prompts Analysis

## Prompt Inventory

- `generate_agent_role_prompt()` in `examples/gpt-researcher/agent/prompts.py`
- `generate_report_prompt()`
- `generate_search_queries_prompt()`
- `generate_resource_report_prompt()`
- `generate_outline_report_prompt()`
- `generate_concepts_prompt()`
- `generate_lesson_prompt()`
- `auto_agent_instructions()`

## Prompt Function Map

- `auto_agent_instructions()` classifies the user task into a domain agent.
- `generate_agent_role_prompt()` establishes the persona and output style for the chosen domain.
- `generate_search_queries_prompt()` converts the user topic into three search queries.
- `generate_resource_report_prompt()` asks for a source recommendation report.
- `generate_outline_report_prompt()` asks for a report outline.
- `generate_concepts_prompt()` asks for five concepts to learn from the research summary.
- `generate_lesson_prompt()` asks for a concept lesson in Markdown.
- `generate_report_prompt()` turns the collected research summary into the final report.

## Prompt Structure

- The prompts are simple string templates rather than a layered prompt system.
- Most prompts follow the same pattern: quoted context, explicit task, output constraints, and a final formatting rule.
- The role prompt is separate from the task prompt, which keeps the domain persona stable while the user task changes.

## Core Prompt Elements

- Role framing: domain-specific personas like finance, travel, academic research, business, and security.
- Task framing: search query generation, report synthesis, outline creation, concept extraction, lesson writing.
- Context injection: the accumulated `research_summary` is embedded directly into later prompts.
- Output constraints: list-of-strings JSON for queries and concepts, Markdown for reports.
- Authority/stance instruction: the final report prompt explicitly says to form a concrete opinion and avoid meaningless conclusions.
- Date anchoring: several prompts inject the current date.

## Function-to-Mechanism Mapping

- Task routing is implemented by `auto_agent_instructions()` plus `choose_agent()` in `examples/gpt-researcher/agent/llm_utils.py`.
- Report generation is implemented by putting the research summary inside a prompt wrapper and asking for a long-form Markdown report.
- Query generation is implemented by a narrow format contract that requests exactly three JSON strings.
- Summary reuse is implemented by passing the accumulated `research_summary` into later prompt builders.

## Prompt Organization Pattern

- The prompt system is organized by workflow stage more than by model role.
- The stage order is: classify agent -> generate search queries -> collect summaries -> generate report -> optionally generate lessons.
- Domain personas are a secondary layer that gets selected before the stage prompts run.

## Prompt Composition Strategy

- Static instruction text is combined with runtime values such as task, question, research summary, and date.
- The report prompt is deliberately verbose and opinionated, while the query prompt is minimal and format-heavy.
- Later prompts reuse earlier summaries instead of reconstructing context from scratch.

## Prompt-to-Code Coupling

- `ResearchAgent.create_search_queries()` calls `generate_search_queries_prompt()`.
- `ResearchAgent.write_report()` maps `report_type` to a specific prompt builder through `get_report_by_type()`.
- `ResearchAgent.write_lessons()` uses `generate_concepts_prompt()` and `generate_lesson_prompt()`.
- `choose_agent()` drives the domain role prompt selection from the initial task.

## Constraint and Reliability Strategy

- The search query prompt constrains output to a JSON list of three strings.
- The concept prompt also expects JSON-like list output.
- The report prompt uses a minimum word count, Markdown syntax, and APA-style source listing.
- Reliability is mostly achieved through repeated summarization and source accumulation, not through strict schema validation.

## Likely Failure Modes

- JSON parsing can fail if the model does not follow the list format exactly.
- The role-selection prompt can misclassify the task and send the run through the wrong persona.
- The final report prompt can produce confident but overly generalized conclusions if the `research_summary` is weak.
- Prompt quality depends heavily on the quality of the accumulated web summaries.

## Reusable Prompt Patterns

- Separate persona selection from task prompts.
- Use a short structured prompt for query generation and a longer prompt for final synthesis.
- Inject accumulated evidence into later stages instead of asking the model to remember it.
- Give explicit formatting contracts when the downstream code expects machine-readable output.

## My Learning Notes

- This project is a good example of prompt staging rather than prompt richness.
- The prompts are functional and clear, but the system relies more on orchestration and summarization than on sophisticated prompt layering.
- The most interesting prompt pattern here is the explicit separation between "search questions" and "final opinionated synthesis".
