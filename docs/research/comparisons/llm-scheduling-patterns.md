# LLM Scheduling Patterns

## Comparison Goal

Compare how projects schedule LLM calls, break tasks into stages, and handle iteration.

## Compared Projects

- [autoresearch](/Users/uynil/projects/agents/docs/research/projects/autoresearch/README.md)
- [DeepSearchAgent-Demo](/Users/uynil/projects/agents/docs/research/projects/DeepSearchAgent-Demo/README.md)
- [gpt-researcher](/Users/uynil/projects/agents/docs/research/projects/gpt-researcher/README.md)

## Dimensions

- call stages
- serial vs parallel execution
- planner or reviewer roles
- retry or fallback behavior
- schema or parser constraints
- fit for task type

## Comparison Table

| Dimension | autoresearch | DeepSearchAgent-Demo | gpt-researcher |
| --- | --- | --- | --- |
| Core stage layout | setup -> baseline -> edit `train.py` -> commit -> run -> parse metrics -> keep or reset | report structure -> first search -> first summary -> reflection query -> reflection summary -> final formatting | choose role -> generate search queries -> parallel search/scrape -> chunk summarization -> final report -> optional lessons |
| Execution style | single long-lived agent loop with external shell commands and git transitions | fully sequential, paragraph by paragraph | planner/executor with parallel source gathering |
| Planner or reviewer shape | one agent implicitly acts as planner, executor, and judge in the same loop | implicit planner at report-structure stage and local reviewer-like reflection stage | explicit planner flavor in task classification and query generation, but little reviewer behavior |
| Iteration style | indefinite greedy keep-or-reset loop over experiments | bounded per-paragraph reflection loop | breadth-first aggregation with chunk summarization, not reflective revision |
| Structured output style | log summary fields plus TSV experiment ledger | heavy JSON schema contracts at most stages | lighter JSON list contracts for queries and concepts, Markdown for reports |
| Reliability shape | fixed evaluator, fixed time budget, git rollback, crash/timeout rules, log redirection | output cleanup, fallback query generation, manual formatting fallback, bounded reflections | parallel source redundancy, chunk summarization, visited URL deduplication, lighter parsing expectations |
| Best fit | unattended autonomous experiment search on a tightly bounded code surface | sectioned deep-research reports that benefit from local revision | broad web research and report synthesis over many sources |

## Shared Patterns

- all three projects avoid fully open-ended chat loops by imposing a small repeatable workflow
- all three depend on intermediate compression artifacts rather than carrying raw execution context forever
- all three are easier to reason about than free-form agent loops because the main transition pattern is explicit

## Key Differences

- `autoresearch` is not a report-writing pipeline at all; it is an autonomous experiment loop whose scheduling state lives mostly in prompt instructions, shell commands, and git state
- `DeepSearchAgent-Demo` spends more model budget on refining one section at a time, while `gpt-researcher` spends more system effort on collecting and compressing many sources quickly
- `DeepSearchAgent-Demo` pushes structure into prompts and parsers; `gpt-researcher` pushes more of the workload into orchestration and chunk processing
- `autoresearch` is the most externally orchestrated of the three, `DeepSearchAgent-Demo` is the most stage-structured in code, and `gpt-researcher` is the most throughput-oriented

## Conclusions

With the current evidence, these three projects suggest three distinct scheduling families:

- `autoresearch`: prompt-and-git-driven autonomous experiment loop
- `DeepSearchAgent-Demo`: section-oriented, reflection-bounded deep-research pipeline
- `gpt-researcher`: breadth-first planner/executor research pipeline

Together they form a useful spectrum:

- externalized orchestration around a coding agent
- code-defined staged workflow with local reflection
- code-defined research aggregation pipeline with parallel collection

## Open Questions

- how stable is `autoresearch`'s prompt-defined state machine in long unattended runs?
- How much does `DeepSearchAgent-Demo` gain from reflection on a live query?
- How much does `gpt-researcher` lose by not having an explicit reviewer or revision loop?
- After live runs, should these remain in one comparison family or split into separate sub-patterns?
