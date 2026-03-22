---
project: self-evolving-agent
source_path: examples/self-evolving-agent
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: verify whether live OpenClaw sessions actually follow the intended light-loop/full-loop control flow
---

# LLM Orchestration

## LLM Call Map

1. The agent loads [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md) as the main skill contract.
2. On OpenClaw, the bootstrap hook injects an additional reminder from [`hooks/openclaw/handler.ts`](/Users/uynil/projects/agents/examples/self-evolving-agent/hooks/openclaw/handler.ts).
3. The skill points the agent to [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md) as the main operating system.
4. The coordinator then routes behavior into narrower modules depending on the current step:
   - pre-task prioritization via [`modules/learning-agenda.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/learning-agenda.md)
   - diagnosis via [`modules/diagnose.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/diagnose.md)
   - post-task reflection via [`modules/reflection.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/reflection.md)
   - capability updates via [`modules/capability-map.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/capability-map.md)
   - training decisions via [`modules/curriculum.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/curriculum.md)
   - learning-state decisions via [`modules/evaluator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/evaluator.md)
   - durable promotion via [`modules/promotion.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/promotion.md)
5. Benchmarks in [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) execute the skill in one model pass, then judge the output in a second model pass.

## Scheduling Pattern

- The runtime shape is not a graph of specialized worker agents.
- It is a single-agent protocol stack with explicit stage routing.
- The central scheduling decision is effort selection:
  - `light loop`
  - `full loop`
- The full loop is a fixed 10-step sequence.
- Outside task execution, there is a second scheduling layer:
  agenda review, which decides which 1-3 capabilities deserve deliberate practice now.

## Structured Output Strategy

- Most modules define Markdown templates rather than JSON schemas.
- The durable outputs are ledger entries, not ephemeral assistant thoughts.
- Important record types include:
  - diagnosis
  - capability update
  - training unit
  - evaluation
  - promotion decision
  - agenda review
- The demos show the intended output shapes and field discipline for each record type.

## Context Handoff

- The handoff mechanism is document-based rather than message-history-based.
- [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md) decides which module governs the current step.
- The persistent handoff artifacts live in `.evolution` ledgers:
  - `LEARNINGS.md`
  - `ERRORS.md`
  - `CAPABILITIES.md`
  - `LEARNING_AGENDA.md`
  - `TRAINING_UNITS.md`
  - `EVALUATIONS.md`
- The hook injects a compact bootstrap reminder so the agent does not need to re-derive the whole operating model from scratch every session.
- Migration from `self-improving-agent` adds a read-only legacy evidence layer rather than rewriting old state into the new schema.

## Reliability Mechanisms

- Cost control:
  default to the light loop and escalate only when evidence justifies the full pipeline.
- Explicit state ladder:
  `recorded -> understood -> practiced -> passed -> generalized -> promoted`.
- Promotion gate:
  transfer proof is required before durable rule persistence.
- Agenda discipline:
  keep only 1-3 active focus capabilities.
- Bootstrap state:
  capability map and learning agenda ship with conservative seeds instead of starting blank.
- Structural validation:
  [`scripts/run-evals.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-evals.py) checks for expected files and protocol text.
- Behavioral benchmarking:
  [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) runs model-in-the-loop candidate and judge passes over benchmark scenarios.

## Strengths

- The control loop is unusually explicit and teachable.
- The project separates incident logging, capability diagnosis, training, evaluation, and promotion instead of collapsing them.
- The hook, bootstrap script, and benchmarks connect the protocol to actual operating conditions.
- The system is careful about overgeneralization and durable memory pollution.

## Limitations

- The system appears cognitively heavy if used at full strength on everyday work.
- Most enforcement is still prompt/protocol based rather than hard-coded.
- The quality of outcomes likely depends on how consistently the agent actually opens and follows the right modules at the right time.
- The benchmark scenarios may still reward template compliance more than genuine behavioral change.
- There is no evidence yet from a live run that the loop stays intact under pressure.

## Questions for Further Research

- How often does a live agent choose the correct loop depth?
- Which module most often gets skipped or under-followed in practice?
- Are the benchmarks discriminating enough to catch shallow output that merely imitates the templates?
- Does the agenda review actually improve task selection and transfer over time?
