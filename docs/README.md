# Agents Research Docs

This directory stores the learning system for studying open-source agent projects in this repository.

## Structure

- `00-system/`: workflow, prompts, checklists, evaluation rules, and prompt iteration history
- `01-projects/`: one folder per project, with `README.md` as the canonical project entry page
- `02-comparisons/`: cross-project comparison docs
- `03-methodology/`: reusable methods for reading agent systems
- `templates/`: reusable document templates

## Current Scope

The documentation system is initialized first. Detailed project research has not started yet.

Current example projects under `examples/`:

- `BettaFish`
- `DeepSearchAgent-Demo`
- `MiroFish`
- `autoresearch`
- `chatwoot`
- `everything-claude-code`
- `gpt-researcher`

## Recommended Workflow

1. Read [00-system/learning-workflow.md](./00-system/learning-workflow.md)
2. Use [00-system/prompt-library.md](./00-system/prompt-library.md)
3. Write or update `01-projects/<project>/README.md`
4. Record prompt quality updates in [00-system/prompt-iteration-log.md](./00-system/prompt-iteration-log.md)
5. Use [00-system/session-close-checklist.md](./00-system/session-close-checklist.md)
6. Promote stable findings into comparisons, indexes, or `AGENTS.md` only when thresholds are met
