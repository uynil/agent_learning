---
project: self-evolving-agent
source_path: examples/self-evolving-agent
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: map authority boundaries more tightly between SKILL.md, coordinator.md, and the modules
---

# Prompts Analysis

## Prompt Inventory

- [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md)
- [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md)
- [`modules/learning-agenda.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/learning-agenda.md)
- [`modules/diagnose.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/diagnose.md)
- [`modules/capability-map.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/capability-map.md)
- [`modules/curriculum.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/curriculum.md)
- [`modules/evaluator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/evaluator.md)
- [`modules/promotion.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/promotion.md)
- [`modules/reflection.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/modules/reflection.md)
- bootstrap reminder in [`hooks/openclaw/handler.ts`](/Users/uynil/projects/agents/examples/self-evolving-agent/hooks/openclaw/handler.ts)
- benchmark prompts in [`benchmarks/suite.json`](/Users/uynil/projects/agents/examples/self-evolving-agent/benchmarks/suite.json)

## Prompt Function Map

- [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md) establishes the overall learning philosophy, closed loop, effort modes, file map, and operating rules.
- [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md) converts that philosophy into a layered operating stack and a concrete step-by-step workflow.
- The modules each own one narrow reasoning function:
  - prioritization
  - diagnosis
  - reflection
  - capability calibration
  - training design
  - learning-state evaluation
  - promotion gating
- The hook prompt is not a full protocol. It is a compressed retrieval cue that reminds the agent to activate the deeper system at bootstrap time.
- The benchmark prompts encode expected user scenarios that test whether the skill can produce the intended structured outputs.

## Prompt Structure

- This project uses a multi-document prompt stack rather than one monolithic prompt.
- The hierarchy appears to be:
  - top-level skill contract
  - system coordinator
  - step-specific modules
  - asset templates
  - hook reminder
- The prompt documents mix:
  - principle statements
  - decision rules
  - escalation triggers
  - output templates
  - anti-patterns

## Core Prompt Elements

- Role framing:
  the agent is treated as a system that should evolve capabilities, not only solve the immediate task.
- Task framing:
  each task is also a learning opportunity or evaluation event.
- Stage framing:
  learning agenda, diagnosis, training, evaluation, and promotion are distinct stages.
- State framing:
  the six-state ladder makes learning progress explicit.
- Escalation logic:
  light loop versus full loop.
- Output contracts:
  Markdown ledger templates with explicit fields.
- Counterexample and transfer requirements:
  many modules force the model to say when a rule should not apply and how it transfers.
- Persistence policy:
  only validated strategies reach durable memory targets like `AGENTS.md`.

## Function-to-Mechanism Mapping

- Prevent "logging = learning" confusion:
  implemented by the six-state evaluation ladder and promotion gate.
- Keep reflective overhead bounded:
  implemented by the light-loop/full-loop split plus escalation triggers.
- Prioritize training instead of flooding memory:
  implemented by the 1-3 item learning agenda cap.
- Diagnose reusable weaknesses rather than isolated incidents:
  implemented by capability-level root-cause labels and recurrence labels.
- Avoid brittle promotion:
  implemented by transfer checks, counterexamples, and minimal durable rule guidance.
- Keep the system sticky across sessions:
  implemented by hook reminders and bootstrapped ledgers.

## Prompt Organization Pattern

- The prompt system is organized by responsibility layer, not only by workflow stage.
- `SKILL.md` provides the contract.
- `coordinator.md` provides the operating system.
- modules provide local protocols.
- assets provide the persistence schema.
- hook text provides low-latency activation.

## Prompt Composition Strategy

- Mostly static documents with explicit cross-references.
- Runtime variability is supplied by the current task, current ledger contents, and current agenda status.
- The benchmark layer then wraps these prompt contracts in concrete scenarios and grading criteria.

## Prompt-to-Code Coupling

- [`hooks/openclaw/handler.ts`](/Users/uynil/projects/agents/examples/self-evolving-agent/hooks/openclaw/handler.ts) hardcodes a subset of the protocol into a bootstrap injection.
- [`scripts/bootstrap-workspace.sh`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/bootstrap-workspace.sh) depends on the asset files as canonical workspace state.
- [`scripts/run-evals.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-evals.py) validates the presence of key prompt phrases, which tightly couples repository integrity to the prompt docs.
- [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) treats skill invocation as the candidate behavior under test.

## Constraint and Reliability Strategy

- Strong conceptual constraints:
  do not confuse recording with mastery.
- Strong state constraints:
  do not skip ladder transitions.
- Strong promotion constraints:
  require transfer before durable persistence.
- Moderate output constraints:
  use templates rather than strict machine-validated schemas.
- Soft execution constraints:
  most compliance is still entrusted to the agent following the protocol.

## Likely Failure Modes

- The agent may imitate the templates without truly updating behavior.
- The multi-document stack may create authority ambiguity if the agent reads only part of it.
- Heavy protocol overhead may cause silent downgrade back to shallow logging habits.
- Hook reminders may be too compressed to preserve the more nuanced parts of the system.
- Because output is mostly Markdown, structured compliance may be harder to validate automatically than in JSON-heavy systems.

## Reusable Prompt Patterns

- Separate top-level philosophy from step-local protocols.
- Use a compressed hook reminder as a retrieval cue, not a full replacement for the main skill.
- Force counterexamples and transfer checks to reduce brittle overgeneralization.
- Treat durable memory as a privileged target with a stricter gate than ordinary logging.
- Use benchmark scenarios to test protocol adherence, not only repository structure.

## Personal Notes

- This project is a strong example of prompt-as-governance-layer.
- Its prompt stack is more like a policy manual plus operating procedures than like a classic task prompt library.
- The most interesting question is not whether the prompts are elegant, but whether this layered protocol actually changes future behavior.
