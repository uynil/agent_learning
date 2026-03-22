# Prompt Organization Patterns

## Goal

Compare how different projects store, shape, and use prompts.

## Comparison Dimensions

- function grouping
- prompt storage location
- organization style: role-based or stage-based
- mechanism placement: inline, shared template, or builder
- coupling with orchestration code
- output constraint style
- reflection support
- reuse potential

## Comparison Table

| Project | Storage | Organization | Constraints | Reflection | Notes |
| --- | --- | --- | --- | --- | --- |
| BettaFish | TBD | TBD | TBD | TBD | not started |
| DeepSearchAgent-Demo | shared prompt constants in one prompts module | strongly stage-based, one prompt per node | heavy JSON schema and JSON-only output rules | explicit reflection prompts in main path | prompt layer is tightly tied to nodes and repair helpers |
| MiroFish | TBD | TBD | TBD | TBD | not started |
| autoresearch | one top-level `program.md` plus project README context | lifecycle-based operations manual rather than prompt-module staging | shell workflow rules, file-scope constraints, log-format expectations, TSV schema examples | no dedicated reflection prompt; keep-or-reset loop is the main revision mechanism | prompt organization is unusually externalized and tightly coupled to git, shell commands, and a single editable file |
| chatwoot | TBD | TBD | TBD | TBD | not started |
| everything-claude-code | TBD | TBD | TBD | TBD | not started |
| gpt-researcher | prompt builder functions in one prompts module | stage-based with a secondary persona layer | lighter JSON constraints for query-like tasks, Markdown for reports | minimal reflection in the main path | prompt system is simpler and depends more on orchestration than on schema-heavy prompts |

## Shared Patterns

- none of the current compared projects uses a deeply nested prompt-fragment system in the main path
- all three projects keep prompt organization easy to locate, even though the storage style differs
- each project aligns prompt organization with its dominant control structure:
  lifecycle, stages, or persona-plus-stages

## Key Differences

- `autoresearch` does not centralize prompts in a code module at all; it centralizes them in a human-editable operational document
- `DeepSearchAgent-Demo` keeps prompt boundaries nearly identical to orchestration node boundaries
- `gpt-researcher` adds a persona-selection layer before stage prompts run
- `DeepSearchAgent-Demo` relies on stricter schema contracts and an explicit reflection branch
- `gpt-researcher` relies on simpler templates and lets summarization/orchestration do more of the work

## Conclusions

The current compared projects already show a useful three-way split:

- lifecycle-based operational prompt document
- stage-based prompts with strong schema enforcement
- stage-based prompts with lighter templates plus role selection
