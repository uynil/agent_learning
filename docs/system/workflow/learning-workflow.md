# Learning Workflow

This file is the single source of truth for the research workflow.

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
    +--> stop -> update project README only
    |
    v
Run Deep Dive / Prompt Audit
    |
    v
Update project docs
    |
    v
Update prompt iteration log
    |
    v
Decide whether any low-frequency docs should change
```

## Workflow Map

```text
README.md
   |
   v
docs/README.md
   |
   v
learning-workflow.md
   |
   +--> prompt-library.md
   +--> session-close-checklist.md
   +--> prompt-smoke-cases.md
   +--> research-index.md
   +--> docs/research/projects/<project>/README.md
   +--> docs/research/comparisons/*
```

## Documentation Rules

Every research note should clearly separate:

- `Observed Facts`: directly supported by code or docs
- `Inferences`: conclusions derived from the observed facts
- `Open Questions`: missing information or uncertain parts
- `Improvement Ideas`: future research or design opportunities

Agent project findings belong in `docs/research/`, especially:

- `docs/research/projects/*`
- `docs/research/comparisons/*`

Learning-system rules do not belong in project notes.

Learning-system workflow, prompts, evaluation, methodology, and templates belong in `docs/system/*`.

## Update Rules

When editing a project note:

1. Update the frontmatter status and `last_updated`
2. Add new facts before rewriting old conclusions
3. Keep uncertain conclusions in `Open Questions`
4. Add a concrete `next_action`

## Project Output Rules

Each project uses one canonical entry page:

- `docs/research/projects/<project>/README.md`

Additional project docs are optional and should only be added when needed:

- `architecture.md`
- `llm-orchestration.md`
- `prompts-analysis.md`
- `notes.md`

## High-Frequency and Low-Frequency Updates

### High-Frequency

Expected on most sessions:

- project `README.md`
- project deep-dive notes, if the session reached that level
- `docs/system/prompts/prompt-iteration-log.md`

### Low-Frequency

Only update when there is enough evidence:

- `AGENTS.md`
- `docs/research/comparisons/*`
- `docs/system/prompts/prompt-smoke-cases.md`
- `docs/system/workflow/research-index.md`

Use this split:

- learning-system experience -> `AGENTS.md`
- agent design / architecture / orchestration / prompt findings -> `docs/research/projects/*` or `docs/research/comparisons/*`

## Blocked or Low-Confidence Sessions

A blocked session still counts as research output if it records:

- current status
- files already checked
- blocker
- working hypothesis
- next resume point

Do not let failed exploration disappear.

## Session Close

Every research round should end with the checklist in [session-close-checklist.md](./session-close-checklist.md).

At close-out, explicitly decide whether a lesson is:

- a learning-system lesson
- a project-specific or cross-project agent lesson

Do not mix those destinations.

## Research Exit Criteria

A project study session is useful when it leaves behind at least one of these:

- a clearer execution flow
- a better prompt pattern
- a stronger comparison point
- a sharper open question for the next round
