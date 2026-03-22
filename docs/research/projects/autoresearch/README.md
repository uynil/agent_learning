---
project: autoresearch
source_path: examples/autoresearch
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: trace one full experiment loop from prompt instructions into git-state transitions, logging, and rollback behavior
---

# autoresearch

## Quick Scan Summary

`autoresearch` is a minimal autonomous ML research harness, not a full in-repo multi-agent framework.

Its core design is:

- freeze the data and evaluation harness in `prepare.py`
- allow the agent to edit only `train.py`
- encode the research loop itself in `program.md`

The interesting part is not orchestration code inside the repository. The interesting part is that orchestration is mostly externalized into prompt instructions plus git operations.

## Files Checked

- `examples/autoresearch/README.md`
- `examples/autoresearch/program.md`
- `examples/autoresearch/prepare.py`
- `examples/autoresearch/train.py`
- `examples/autoresearch/pyproject.toml`

## Observed Facts

- The project README explicitly says only three files matter for the research loop:
  - `prepare.py` for fixed constants, data prep, dataloader, and evaluation
  - `train.py` for all model and optimization changes
  - `program.md` for agent instructions
- `prepare.py` fixes the main comparability constraints:
  - `TIME_BUDGET = 300`
  - `MAX_SEQ_LEN = 2048`
  - `evaluate_bpb(...)` is marked `DO NOT CHANGE`
- `prepare.py` owns the tokenizer, data download path, BOS-aligned dataloader, and BPB evaluation logic, so the experiment agent does not control the training data pipeline or the metric implementation.
- `train.py` is a single-file GPT training script with model definition, optimizer, training loop, logging, and final summary output all in one place.
- `train.py` exposes its experiment surface mostly as top-level constants:
  - architecture knobs such as `DEPTH`, `ASPECT_RATIO`, `HEAD_DIM`, `WINDOW_PATTERN`
  - optimization knobs such as `TOTAL_BATCH_SIZE`, `MATRIX_LR`, `WEIGHT_DECAY`, `ADAM_BETAS`
  - runtime knob `DEVICE_BATCH_SIZE`
- `train.py` stops based on the fixed wall-clock budget from `prepare.py`, then runs a final validation pass and prints a compact summary including `val_bpb`, `peak_vram_mb`, total tokens, steps, and parameter count.
- `program.md` defines the actual autonomous workflow:
  - create a fresh `autoresearch/<tag>` branch
  - read the in-scope files
  - verify cached data exists
  - initialize `results.tsv`
  - run a baseline experiment first
  - repeat commit -> run -> read metrics -> log -> keep or reset
- `program.md` requires that `results.tsv` stay untracked and that losing experiments should be reverted with git rather than accumulated in the codebase.
- The README markets the project as an autonomous research org or swarm, but the default shipped setup is one agent operating on one mutable file under one fixed evaluation harness.

## Inferences

- The real orchestration layer is prompt-driven, not code-driven. `program.md` acts like a lightweight research operating system for a general-purpose coding agent.
- The repo is intentionally optimized for search-space control:
  - one mutable training file
  - one fixed evaluator
  - one fixed time budget
  - one scalar target metric
  This keeps experiments comparable and failures easy to reason about.
- Git is part of the system design, not just a developer convenience. Branching, committing, and resetting are the mechanism for state advancement, rollback, and experiment selection.
- This project is closer to "autonomous local hill-climbing over a constrained training script" than to a rich deep-research agent with decomposition, reflection, retrieval, or multi-role coordination.
- The "swarm" framing is currently more narrative and extensibility direction than implemented default architecture.
- Because the time budget is machine-local wall clock, the system is strong at within-machine comparison but weak at cross-machine benchmark comparability.

## Open Questions

- How stable is agent behavior over many experiment rounds when the only explicit control plane is `program.md` plus the local git state?
- Does `results.tsv` provide enough memory for long runs, or does the setup need a richer summary and hypothesis log to avoid repeated failed ideas?
- How often do agents violate the "only edit `train.py`" constraint in practice, and how much enforcement is needed beyond instruction text?
- How much of the observed progress comes from model-search quality versus simply exploiting the fixed five-minute budget on a specific GPU?
- What are the practical failure modes around crashes, silent regressions, or poor experiment selection once the easy hyperparameter wins are exhausted?

## Improvement Ideas

- Deep-dive the prompt-to-mechanism mapping in `program.md`:
  - how setup, logging, rollback, and autonomy are each enforced through prompt wording
- Trace one full experiment loop against actual git state and log artifacts to understand the operational state machine, not just the intended one.
- Compare `autoresearch` with projects that implement deeper research loops in code:
  - question rewrite
  - decomposition
  - evidence accumulation
  - reflection
  - resumable memory
- Study whether the design would benefit from a thin coded supervisor that enforces branch hygiene, timeout handling, result logging, and rollback invariants instead of leaving all of that to the agent prompt.

## Related Notes

- [`llm-orchestration.md`](./llm-orchestration.md)
- [`prompts-analysis.md`](/Users/uynil/projects/agents/docs/research/projects/autoresearch/prompts-analysis.md)
