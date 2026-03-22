---
project: Multi-Agent-Data-Analyst
source_path: examples/Multi-Agent-Data-Analyst
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare the inline prompt patterns against a live run and see which prompts actually dominate behavior
---

# Prompts Analysis

## Prompt Inventory

- `app/agent_orchestrator.py`
  - orchestrator system prompt
  - tool contracts for `sql_agent`, `visualization_agent`, and `determine_data_source`
- `app/tools/sql_data_analyst_agent.py`
  - SQL query generation prompt
  - SQL executor-wrapper prompt
  - Plotly visualization prompt
- `app/tools/data_analyst_agent.py`
  - default visualization system prompt
  - DataFrame summary + user instruction prompt wrapper

There is no dedicated prompt library file in the project. The prompt layer is inline, local to the code that uses it.

## Prompt Structure

The dominant structure is:

1. role framing
2. task instruction
3. constraints
4. output format
5. example or implied executable contract

Examples:

- The orchestrator prompt states the role of the agent and the two tool choices.
- The SQL prompt supplies database schema text, the user question, and a strict "return only SQL" contract.
- The function-generation prompt converts the SQL into a callable Python function with a fixed name.
- The visualization prompt injects a sample of the result table and asks for a Plotly function.
- The DataVisualizationAgent prompt adds a JSON response example with `code` and `explanation`.

## Core Prompt Elements

- Role assignment: "expert data analysis orchestrator", "expert SQL developer", "expert data visualization AI"
- Context injection: schema, sample rows, DataFrame summary, user instructions
- Hard output constraints: "return ONLY the SQL query", "return ONLY the Python code", JSON with two keys
- Named artifacts: `execute_sql_query`, `create_visualization`
- Example-shaped guidance: especially the JSON example in `DataVisualizationAgent`
- Minimal reflection: retry happens in code, not in a prompt-level self-critique loop

## Function-to-Mechanism Mapping

- Orchestrator system prompt
  - Function: route the request to the right capability
  - Mechanism: role framing + tool naming + simple task breakdown

- SQL generation prompt
  - Function: synthesize a query from a user question and live schema
  - Mechanism: schema injection + strict SQL-only output rule

- SQL executor-wrapper prompt
  - Function: turn the query into a callable Python function
  - Mechanism: code template with a fixed function name and connection contract

- Visualization prompt inside `SQLDataAnalysisAgent`
  - Function: generate a chart from tabular query results
  - Mechanism: result sample + plotting constraints + code-only output

- `DataVisualizationAgent` system prompt
  - Function: produce reusable Plotly code from a DataFrame
  - Mechanism: role framing + JSON output schema + worked example

## Prompt-to-Code Coupling

The prompt outputs are expected to be executable artifacts, not prose explanations.

- SQL strings are executed directly with SQLAlchemy.
- Generated Python functions are executed with `exec()`.
- Plotly code is executed with `exec()` after extracting from markdown or JSON.
- The code only does shallow sanity checks like "does the string contain the expected function name?"

This means the prompt design is inseparable from runtime safety. A bad prompt output is not just a bad answer; it can become a runtime failure.

## Reliability and Failure Modes

- JSON parsing can fail, after which the code falls back to regex extraction.
- Python code blocks can fail, after which the code retries generation.
- SQL execution can fail, after which the system tries a generated Python wrapper.
- Visualization generation can fail if no `Figure` object is produced.
- There is no prompt versioning, no externalized prompt registry, and no separate prompt evaluation harness.

## Inferences

- The prompt layer is intentionally lean and highly local rather than centrally managed.
- Most of the real control comes from prompt shape plus code-level enforcement, not from a richer agent framework.
- Prompt quality probably matters most in the SQL and visualization branches, because those branches directly synthesize executable code.

## Open Questions

- Would extracting these inline prompts into a shared prompt file reduce drift or just add another maintenance surface?
- Does the model reliably honor the JSON and code-only contracts when run live?
- Would a stronger parser or schema validator improve stability more than prompt refinement alone?

