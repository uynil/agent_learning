---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare state persistence with a live interrupted run
---

# Context Management

## Context Sources

- user query
- report structure titles and descriptions
- current paragraph content
- search query generated for the current round
- search results truncated for prompt input
- paragraph latest summary
- search history
- reflection iteration count
- saved state JSON

## Memory and State Layers

- `State` is the top-level memory container.
- `Paragraph` is the unit of local research context.
- `Research.latest_summary` is the working memory slot that gets handed from one stage to the next.
- `Research.search_history` is the durable evidence log.
- `Research.reflection_iteration` tracks how many refinement passes have happened.
- `State.save_to_file()` persists the whole tree.
- `State.load_from_file()` restores the tree for later continuation.

## Retrieval and Selection Strategy

- There is no semantic retrieval system or vector store.
- Context selection is explicit and manual.
- The agent passes only the current paragraph and its latest summary into the next stage.
- Search results are selected by Tavily and then truncated before being inserted into prompts.
- The system uses scope reduction rather than retrieval ranking.

## Context Packing Strategy

- `format_search_results_for_prompt()` truncates search result content to fit the prompt budget.
- The stage prompts carry only the fields needed for that step.
- The code avoids sending the entire research history back into every call.
- Paragraph-level summaries compress the search trail into a smaller working context.

## Context Update Rules

- `FirstSummaryNode` writes the initial `latest_summary`.
- `ReflectionSummaryNode` replaces that summary with a revised version.
- `ReflectionSummaryNode` increments `reflection_iteration`.
- Search results are appended to `search_history` after each search pass.
- The state timestamp is updated whenever paragraph data changes.
- A completed paragraph is marked complete only after the reflection loop finishes.

## Failure Modes

- The paragraph summary can become too compressed and lose nuance.
- The search history can grow without improving the next prompt if the summary is weak.
- There is no deduplication or explicit forgetting policy.
- Context is local to a paragraph, so global cross-section synthesis is limited.
- Manual resume is possible, but there is no first-class resume workflow in the main agent loop.

## Reusable Patterns

- keep the working context narrow
- preserve a resumable summary instead of the whole transcript
- separate archival evidence from working memory
- make context handoff explicit between rounds
- save state to disk when a session can be interrupted
