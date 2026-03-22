---
project: self-evolving-agent
source_path: examples/self-evolving-agent
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect how much of the ledger state is truly consulted during live task execution versus only written after the fact
---

# Context Management

## Context Sources

- current user task
- active learning agenda
- prior learnings
- prior errors
- capability map
- open training units
- evaluations
- legacy self-improving logs when migration exists
- hook reminder at bootstrap

## Memory and State Layers

- `LEARNINGS.md` stores reusable lessons and trigger signatures.
- `ERRORS.md` stores failure evidence and initial diagnostic hypotheses.
- `FEATURE_REQUESTS.md` stores desired capabilities not yet reliably achieved.
- `CAPABILITIES.md` stores current capability levels, limits, evidence, and upgrade conditions.
- `LEARNING_AGENDA.md` stores the active 1-3 focus capabilities.
- `TRAINING_UNITS.md` stores deliberate practice units.
- `EVALUATIONS.md` stores state transitions on the learning ladder.
- `legacy-self-improving/` stores imported read-only historical logs when migration has happened.

## Retrieval and Selection Strategy

- Retrieval is explicit rather than automatic.
- Before meaningful work, the system tells the agent to retrieve:
  - related learning entries
  - related errors
  - matching capabilities
  - active agenda items
  - open training units
  - legacy logs if they exist
- The learning agenda acts as a priority filter so not every weakness enters working context.
- Trigger signatures in learnings and training units are intended to make later retrieval more timely.

## Context Packing Strategy

- The system avoids one giant memory file by splitting state across specialized ledgers.
- The hook reminder compresses the whole operating model into a short bootstrap cue.
- Light-loop mode further compresses working context by limiting retrieval to the most relevant 1-3 items.
- Capability-focused summaries in the ledgers are designed to carry forward the useful abstraction, not the whole raw incident transcript.

## Context Update Rules

- After meaningful tasks, incidents can be logged into `ERRORS.md` and `LEARNINGS.md`.
- Diagnosis updates the capability view and may create a training unit.
- Evaluations update the learning state ladder.
- Agenda review can reprioritize the active focus set.
- Promotion can move a validated rule into durable context targets outside `.evolution`, such as `AGENTS.md` or `TOOLS.md`.
- The migration layer is intentionally append-only and read-only unless an item is later normalized into the new system.

## Failure Modes

- Ledger sprawl:
  there are many ledgers, so the system may become write-heavy but retrieval-light.
- Retrieval miss:
  relevant prior lessons may still exist but not be surfaced in time.
- Template compliance without behavior change:
  ledgers can look healthy while execution habits stay weak.
- Agenda drift:
  too many active focuses would dilute the intended leverage advantage.
- Promotion pollution:
  if transfer evidence is judged too loosely, durable memory can still be contaminated.

## Reusable Patterns

- separate incident memory from capability state
- keep an explicit active agenda instead of training everything at once
- use trigger signatures to support future retrieval
- preserve legacy memory losslessly during schema migration
- gate durable memory with stricter evidence than ordinary logging

## Comparison Notes

- Compared with `gpt-researcher`, this system has much richer explicit persistent state.
- Compared with `DeepSearchAgent-Demo`, the state is less about one task's evolving content and more about the agent's evolving competence across tasks.
- Compared with `autoresearch`, the state is semantic and pedagogical rather than operational and git-centered.
