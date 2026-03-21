# Prompt Smoke Cases

Use these cases after meaningful edits to `prompt-library.md`.

The goal is not exhaustive evaluation. The goal is a stable manual regression baseline.

## Case 1: New Project Quick Scan

- Target: a project not previously studied in depth
- Prompt path: `Master Prompt` -> `Quick Scan Prompt`
- Expected qualities:
  - identifies a plausible research mode
  - points to real files
  - separates facts from guesses

## Case 2: Workflow-Heavy Project

- Target: a project with staged or graph-like orchestration
- Prompt path: `Architecture Prompt` + `LLM Orchestration Prompt`
- Expected qualities:
  - distinguishes stages clearly
  - explains data flow
  - calls out reliability mechanisms or missing ones

## Case 3: Prompt-Centric Project

- Target: a project where prompts are first-class design artifacts
- Prompt path: `Prompt Audit Prompt`
- Expected qualities:
  - inventories prompt locations
  - explains prompt-to-code coupling
  - identifies likely failure modes

## Case 4: Cross-Project Comparison

- Target: two projects with clearly different orchestration styles
- Prompt path: `Master Prompt` -> `Cross Project Compare`
- Expected qualities:
  - compares by dimension, not by vague impression
  - names tradeoffs
  - cites evidence from both projects

## Case 5: Blocked Research Session

- Target: a messy or low-signal project
- Prompt path: `Quick Scan Prompt` + `Doc Refine Prompt`
- Expected qualities:
  - admits uncertainty
  - records blocker and resume point
  - avoids fake confidence

## Review Method

For each smoke case, score output using [evaluation-rubric.md](./evaluation-rubric.md).

If a prompt change makes two or more smoke cases materially worse, do not promote the change into stable workflow guidance yet.

