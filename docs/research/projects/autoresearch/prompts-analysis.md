---
project: autoresearch
source_path: examples/autoresearch
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: test whether the prompt contract is robust to format drift in training logs and long experimental sessions
---

# Prompts Analysis

## Prompt Inventory

- [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md)
- [`README.md`](/Users/uynil/projects/agents/examples/autoresearch/README.md) as surrounding project-context prompt material

## Prompt Function Map

- [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md) defines setup, experiment scope, logging rules, crash handling, and stop conditions.
- Its setup section aligns the agent with the repo state before experiments begin.
- Its experimentation section narrows the editable surface to [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py) and the target metric to `val_bpb`.
- Its output and logging sections convert raw script output into a compact memory format the agent can reuse.
- Its final loop section turns a general coding agent into an autonomous optimizer that should continue until interrupted.

## Prompt Structure

- The prompt is a single static Markdown document with strong sectioning:
  - setup
  - experimentation
  - output format
  - logging results
  - experiment loop
- The instruction order follows the real lifecycle of a run, which reduces ambiguity.
- It mixes rules, concrete shell commands, examples, and policy statements instead of relying only on abstract goals.

## Core Prompt Elements

- Role framing:
  the agent is treated as an autonomous researcher.
- Task framing:
  improve model quality under a fixed five-minute experiment budget.
- Scope restriction:
  modify only [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py); do not modify [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py) or dependencies.
- Objective:
  minimize `val_bpb`.
- Secondary criteria:
  keep VRAM growth reasonable and prefer simpler wins over ugly complexity.
- Output contracts:
  rely on the final training summary and on a strict TSV schema.
- Persistence policy:
  `NEVER STOP` until manually interrupted.

## Function-to-Mechanism Mapping

- Setup correctness is implemented through explicit preflight steps:
  branch creation, file reads, cache verification, and `results.tsv` initialization.
- Search-space control is implemented through a direct prohibition:
  only [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py) may be edited.
- Metric discipline is implemented by naming [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py) and `evaluate_bpb(...)` as non-editable ground truth.
- Baseline anchoring is implemented by an unconditional first-run rule.
- Context control is implemented by redirecting the whole run to `run.log`, then extracting only the key lines instead of keeping live console output in context.
- Long-run autonomy is implemented by the emphatic `LOOP FOREVER` and `NEVER STOP` language.
- Experiment selection is implemented by a simple branch-advance versus git-reset rule.
- Experiment memory is implemented by the TSV schema and example rows.

## Prompt Organization Pattern

- The prompt is organized by lifecycle stage, not by abstract capability.
- The most important instructions appear exactly where the agent needs them:
  constraints before editing, output schema before parsing, rollback rule inside the loop.
- Examples are used only where parsing or logging could drift:
  summary output and TSV rows.

## Prompt Composition Strategy

- The document is mostly static.
- Runtime variability is left to the agent:
  today's branch tag, specific experiment descriptions, specific code changes, and the evolving current-best commit.
- This keeps the prompt lightweight, but also means the agent itself must supply most of the hypothesis generation.

## Prompt-to-Code Coupling

- [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md) depends on the exact final summary fields printed in [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py).
- The "do not modify evaluator" rule depends on the separation of responsibilities in [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py).
- The prompt's keep/discard logic assumes git is available and that the working tree can be cleanly reset after bad experiments.
- The prompt's setup phase depends on cache layout under `~/.cache/autoresearch/`, which is also defined in [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py).

## Constraint and Reliability Strategy

- Hard constraints:
  editable file boundary, evaluator immutability, no new dependencies.
- Soft constraints:
  VRAM growth and simplicity preference.
- Parsing reliability:
  grep fixed summary keys from `run.log`.
- Failure handling:
  treat missing summary output as crash, inspect last 50 log lines, and give up after a few failed repair attempts.
- Human-availability assumption:
  the prompt explicitly assumes the user may be asleep, so the agent must not seek repeated approval.

## Likely Failure Modes

- If [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py) changes its summary format, the grep-based parser in the prompt may silently fail.
- If the agent resets to the wrong commit, the branch history and `results.tsv` can diverge.
- Tiny metric changes may be over-trusted because the prompt does not define a noise threshold or rerun policy.
- The prompt asks for indefinite autonomy but does not provide a richer hypothesis ledger, which may cause repeated or low-quality experiments later in long runs.
- The instruction "from current master" may drift if the repo's default branch changes.

## Reusable Prompt Patterns

- Keep the mutable code surface tiny and explicit.
- Treat evaluation as a fixed harness outside the agent's authority.
- Convert noisy execution output into compact, grep-friendly summaries.
- Use git state itself as the keep/discard memory mechanism.
- Encode "do not wait for the human" directly when long unattended execution is the intended mode.

## Personal Notes

- This is a strong example of prompt-as-operations-manual rather than prompt-as-reasoning-trick.
- The most interesting design move is not a clever phrase; it is the way the prompt delegates memory, parsing, and rollback to ordinary developer tools.
- For research-system study, `autoresearch` is valuable precisely because it shows how far a simple static prompt can go before coded orchestration becomes necessary.
