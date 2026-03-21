# Learning Workflow

This file is the single source of truth for the research workflow.

## Goal

Build a reusable research workflow for understanding open-source agent projects from several connected angles:

- architecture design
- LLM orchestration and scheduling
- prompt structure, core elements, organization, and usage
- context management, memory, and retrieval strategy
- deep research loops such as question rewrite, decomposition, and evidence accumulation
- reusable lessons for future projects

## Core Research Themes

Use the workflow to study these themes explicitly, not only the main execution path:

- architecture and module boundaries
- orchestration and role splits
- prompt structure and core elements
- prompt function-to-mechanism mapping and how prompts implement specific functions
- prompt reliability and failure modes
- context management and memory strategy
- deep research mode and multi-round investigation design
- evaluation and stopping conditions

## Research Modes

### 1. Quick Scan

Use this first to answer:

- what problem the project solves
- how the project is organized
- whether it is worth a deeper study
- which files deserve the next read

### 2. Deep Dive

Use this when a project is worth detailed study. Focus on execution flow, state transitions, tool use, and prompt-to-code coupling.

### 3. Prompt Audit

Use this when the main target is understanding prompt design, prompt structure, output constraints, prompt layout, and prompt failure modes.

### 4. Context Management Audit

Use this when the main target is understanding how a project manages memory, retrieval, context packing, summaries, and state handoff across steps or rounds.

### 5. Deep Research Mode

Use this when the project appears to support broad-question investigation through query rewrite, decomposition, search-read-synthesize loops, reflection, or iterative summarization.

### 6. Cross Project Compare

Use this after at least two projects have notes. Focus on recurring patterns, differences, tradeoffs, and reusable abstractions.

### 7. Doc Refine

Use this after a study session to turn raw notes into stable docs that can be updated later.

## Standard Session Flow

```text
Select project
    |
    v
Run Quick Scan
    |
    v
Choose primary research theme
    |
    +--> stop early -> update project README only
    |
    +--> Prompt Audit
    |
    +--> Context Management Audit
    |
    +--> Deep Research Mode
    |
    v
Run Deep Dive / focused audit
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

## Deep Research Loop

When the project is complex or the main question is still too broad, use a round-based loop instead of one long pass.

```text
Start with one broad research question
    |
    v
Rewrite it into a sharper question
    |
    v
Pick the smallest useful file set
    |
    v
Extract evidence, contradictions, and gaps
    |
    v
Write a round summary and resumable context
    |
    v
Decide: stop, continue, or pivot
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

## Context Management Rules

When studying a project with long flows or multiple rounds, keep the working context deliberately small.

- keep one active research question per round
- keep a short list of files checked in the current round
- preserve concrete evidence, not whole transcripts
- maintain a resumable context summary instead of carrying raw chat history
- record terminology, state objects, and unresolved gaps that the next round must inherit

Treat context management itself as a research topic:

- what enters each model call
- what survives across steps or rounds
- what is summarized, retrieved, cached, or dropped
- what failure modes come from context overload or context loss

## Deep Research Output Rules

If a session enters `Deep Research Mode`, each round should leave behind:

- current core question
- rewritten research question
- files checked in this round
- observed facts
- current hypothesis
- disconfirmed ideas, if any
- next round question
- resumable context summary

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
- `context-management.md`
- `deep-research-mode.md`
- `prompts-analysis.md`
- `notes.md`

Use `notes.md` for round summaries when the session spans multiple questions or multiple passes.

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
