---
project: autoresearch
source_path: examples/autoresearch
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: validate the intended git-state machine against one real experimental run
---

# LLM Orchestration

## LLM Call Map

1. The human points a general-purpose coding agent at [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md) and tells it to start an experiment, as described in [`README.md`](/Users/uynil/projects/agents/examples/autoresearch/README.md).
2. The agent reads the small in-scope file set named in [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md): [`README.md`](/Users/uynil/projects/agents/examples/autoresearch/README.md), [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py), and [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py).
3. The agent performs setup actions in the shell:
   - create a fresh `autoresearch/<tag>` branch
   - verify cached data
   - initialize `results.tsv`
4. The first experiment is always the untouched baseline run.
5. After that, one external agent repeatedly plays planner, executor, and judge in the same loop:
   - edit [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py)
   - commit
   - run training
   - read summary metrics from `run.log`
   - decide keep versus reset

## Scheduling Pattern

- The default system is single-agent, not multi-role.
- Planning, implementation, execution, failure handling, and result judgment are collapsed into one long-lived agent loop.
- The schedule has two phases:
  - setup gate
  - infinite best-improvement loop
- The keep/discard decision is local and greedy:
  - if `val_bpb` improves, advance the branch
  - otherwise reset to the prior commit

## Context Handoff

- The main cross-iteration state is not an in-memory orchestration object. It is spread across:
  - current git branch and commit
  - current contents of [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py)
  - `results.tsv`
  - `run.log`
- [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md) explicitly keeps training output out of the live context window by requiring `uv run train.py > run.log 2>&1`, then extracting only summary fields with `grep` or failure traces with `tail`.
- This means context handoff is file-based and command-based rather than message-history-based.

## Structured Output Strategy

- [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py) prints a fixed summary block at the end of every successful run, including:
  - `val_bpb`
  - `training_seconds`
  - `peak_vram_mb`
  - `num_steps`
  - `num_params_M`
- [`program.md`](/Users/uynil/projects/agents/examples/autoresearch/program.md) couples directly to that summary format by telling the agent to grep `val_bpb` and `peak_vram_mb`.
- Experiment history is normalized into a five-column TSV:
  - `commit`
  - `val_bpb`
  - `memory_gb`
  - `status`
  - `description`

## Reliability Mechanisms

- Fixed evaluator:
  [`prepare.py`](/Users/uynil/projects/agents/examples/autoresearch/prepare.py) owns `evaluate_bpb(...)` and marks it as the non-editable ground-truth metric.
- Fixed budget:
  training time is anchored to `TIME_BUDGET = 300`.
- Search-space restriction:
  the agent is told to edit only [`train.py`](/Users/uynil/projects/agents/examples/autoresearch/train.py).
- Baseline-first rule:
  the first run must be the unmodified baseline.
- Git rollback:
  losing experiments are discarded by reset instead of silently accumulating.
- Timeout and crash rules:
  runs above 10 minutes or empty grep output are treated as failures.
- Context-throttling rule:
  redirect all output to `run.log` and read only the few lines needed.

## Strengths

- The orchestration is extremely small and understandable.
- The experiment loop is auditable because each candidate is a commit and each decision is reflected in git state plus `results.tsv`.
- Prompt logic and code logic are tightly aligned:
  the prompt only asks the agent to manipulate surfaces that the code actually exposes.
- The design keeps the agent focused on local improvement under a stable evaluator, which reduces a lot of typical autonomous-agent complexity.

## Limitations

- Almost all enforcement is social rather than programmatic. If the agent ignores the prompt, the repo itself does not strongly stop it.
- There is no distinct judge model, retry controller, or reflection stage beyond what one agent improvises for itself.
- Memory is thin:
  `results.tsv` stores outcomes, but not hypotheses, failure clusters, or richer experiment lineage.
- The orchestration is tightly coupled to exact shell behavior and exact log strings, so small output-format changes could break the loop.
- The current design is local hill-climbing, not broad deep research with decomposition, retrieval, or evidence synthesis.

## Questions for Further Research

- In real runs, how often does the agent actually maintain the intended keep/reset invariant without drift?
- Does the greedy keep-only-if-better policy prevent useful exploratory detours?
- How much variance exists in `val_bpb` across repeated identical runs on the same machine?
- Would a thin coded supervisor improve robustness without losing the current system's simplicity?
