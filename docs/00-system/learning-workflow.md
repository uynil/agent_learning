# Learning Workflow

## Goal

Build a reusable research workflow for understanding open-source agent projects from four angles:

- architecture design
- LLM orchestration and scheduling
- prompt design and usage
- reusable lessons for future projects

## Research Modes

### 1. Quick Scan

Use this first to answer:

- what problem the project solves
- how the project is organized
- whether it is worth a deeper study
- which files deserve the next read

### 2. Deep Dive

Use this when a project is worth detailed study. Focus on execution flow, state transitions, tool use, and prompt-to-code coupling.

### 3. Cross Project Compare

Use this after at least two projects have notes. Focus on recurring patterns, differences, tradeoffs, and reusable abstractions.

### 4. Prompt Audit

Use this when the main target is understanding prompt design, output constraints, prompt layout, and prompt failure modes.

### 5. Doc Refine

Use this after a study session to turn raw notes into stable docs that can be updated later.

## Standard Session Flow

```text
Select project
    |
    v
Run Quick Scan
    |
    v
Decide: stop or go deeper
    |
    +--> stop -> update project overview only
    |
    v
Run Deep Dive / Prompt Audit
    |
    v
Update project docs
    |
    v
Update comparison docs
    |
    v
Update prompt iteration log
```

## Documentation Rules

Every research note should clearly separate:

- `Observed Facts`: directly supported by code or docs
- `Inferences`: conclusions derived from the observed facts
- `Open Questions`: missing information or uncertain parts
- `Improvement Ideas`: future research or design opportunities

## Update Rules

When editing a project note:

1. Update the frontmatter status and `last_updated`
2. Add new facts before rewriting old conclusions
3. Keep uncertain conclusions in `Open Questions`
4. Add a concrete `next_action`

## Research Exit Criteria

A project study session is useful when it leaves behind at least one of these:

- a clearer execution flow
- a better prompt pattern
- a stronger comparison point
- a sharper open question for the next round

