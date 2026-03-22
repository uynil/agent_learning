---
project: self-evolving-agent
source_path: examples/self-evolving-agent
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: run the remaining benchmark scenarios through the new `chat/completions` backend and compare the results with the original `codex` path
---

# self-evolving-agent

## Quick Scan Summary

`self-evolving-agent` is not primarily a task-execution agent.

It is a meta-learning skill system for OpenClaw and coding agents that tries to turn passive self-improvement into a closed-loop capability-evolution process.

Its core design is:

- treat incidents as evidence, not as learning completion
- maintain explicit capability state and a small learning agenda
- generate training units for recurring or high-leverage weaknesses
- require evaluation and transfer before promotion into durable policy

This makes it structurally different from the research-agent projects already studied in this repo.
It is closer to an agent operating system layer for learning and self-calibration than to a web-research or experiment-execution pipeline.

## Files Checked

- `examples/self-evolving-agent/README.md`
- `examples/self-evolving-agent/SKILL.md`
- `examples/self-evolving-agent/system/coordinator.md`
- `examples/self-evolving-agent/modules/learning-agenda.md`
- `examples/self-evolving-agent/modules/diagnose.md`
- `examples/self-evolving-agent/modules/capability-map.md`
- `examples/self-evolving-agent/modules/curriculum.md`
- `examples/self-evolving-agent/modules/evaluator.md`
- `examples/self-evolving-agent/modules/promotion.md`
- `examples/self-evolving-agent/modules/reflection.md`
- `examples/self-evolving-agent/assets/*.md`
- `examples/self-evolving-agent/hooks/openclaw/HOOK.md`
- `examples/self-evolving-agent/hooks/openclaw/handler.ts`
- `examples/self-evolving-agent/scripts/bootstrap-workspace.sh`
- `examples/self-evolving-agent/scripts/migrate-self-improving.py`
- `examples/self-evolving-agent/scripts/run-evals.py`
- `examples/self-evolving-agent/scripts/run-benchmark.py`
- `examples/self-evolving-agent/tests/test_run_benchmark.py`
- `examples/self-evolving-agent/benchmarks/suite.json`
- `examples/self-evolving-agent/evals/evals.json`
- `examples/self-evolving-agent/agents/openai.yaml`

## Observed Facts

- The project README explicitly frames the system as an upgrade from `self-improving-agent`:
  - incident logging -> capability evolution
  - passive memory -> active learning agenda
  - correction archive -> curriculum + evaluation + promotion gate
- [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md) defines a 10-step closed loop:
  classify task -> retrieve memory -> pre-task diagnosis -> choose strategy -> perform task -> reflect -> update capability map -> create training -> evaluate -> promote.
- [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md) explicitly separates six learning states:
  `recorded -> understood -> practiced -> passed -> generalized -> promoted`.
- [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md) organizes the system into layered subsystems:
  learning agenda, memory, diagnosis, training, and evaluation/policy.
- The coordinator distinguishes a `light loop` from a `full loop` and defines escalation triggers from the lighter mode into the full pipeline.
- The modules are narrow and role-specific:
  - `learning-agenda.md` prioritizes 1-3 capabilities at a time
  - `diagnose.md` turns incidents into capability-level root-cause analysis
  - `capability-map.md` maintains levels, limits, evidence, and upgrade conditions
  - `curriculum.md` defines training units with drills, pass criteria, and transfer scenarios
  - `evaluator.md` enforces learning-state transitions
  - `promotion.md` requires transfer proof before durable promotion
  - `reflection.md` forces self-explanation, counterexample, and trigger signature
- The assets directory is not just logging storage. It contains separate persistent ledgers for:
  - learnings
  - errors
  - feature requests
  - capability map
  - learning agenda
  - training units
  - evaluations
- [`assets/CAPABILITIES.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/assets/CAPABILITIES.md) ships with conservative bootstrap capability entries and a 5-level rubric from `L1 aware` to `L5 transferable`.
- [`assets/LEARNING_AGENDA.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/assets/LEARNING_AGENDA.md) ships with a bootstrap active focus set rather than leaving the agenda blank.
- [`hooks/openclaw/HOOK.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/hooks/openclaw/HOOK.md) and [`hooks/openclaw/handler.ts`](/Users/uynil/projects/agents/examples/self-evolving-agent/hooks/openclaw/handler.ts) inject a reminder at `agent:bootstrap`, so the capability-evolution framing enters the agent before substantial work begins.
- [`scripts/bootstrap-workspace.sh`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/bootstrap-workspace.sh) initializes a `.evolution` workspace by copying all ledger templates and optionally migrating legacy `.learnings/`.
- [`scripts/migrate-self-improving.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/migrate-self-improving.py) preserves legacy logs losslessly in `.evolution/legacy-self-improving/` instead of rewriting them into the new schema.
- [`scripts/run-evals.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-evals.py) performs structural compliance checks over repository contents and expected phrases.
- [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) runs model-in-the-loop benchmarks by invoking `codex exec` for both candidate and judge passes.
- [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) now also supports a `--backend chat-completions` mode:
  it posts directly to a `chat/completions` endpoint, injects [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md) and [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md) for candidate runs, and carries the no-tools execution contract through the candidate prompt.
- [`benchmarks/suite.json`](/Users/uynil/projects/agents/examples/self-evolving-agent/benchmarks/suite.json) defines four benchmark scenarios:
  pre-task risk diagnosis, post-task diagnosis and training, evaluation and promotion, and agenda review.
- Running `python3 scripts/run-evals.py examples/self-evolving-agent` in this repo state on 2026-03-21 passed 12/12 structural checks.
- Running `python3 -m unittest examples/self-evolving-agent/tests/test_run_benchmark.py` in this repo state on 2026-03-21 passed 4/4 tests.
- Those tests cover:
  - decoding timeout-path `stdout` and `stderr` bytes without crashing
  - candidate prompt insertion of the explicit no-tools contract
  - candidate `chat/completions` message construction with injected skill context
  - judge prompt insertion of an explicit JSON-output contract
- Running a one-scenario smoke test through `python3 scripts/run-benchmark.py --max-scenarios 1 --candidate-model gpt-5.4-mini --judge-model gpt-5.4-mini --timeout-seconds 90` did not complete successfully:
  the candidate call timed out and the timeout handler then crashed with a `TypeError` when concatenating bytes and strings.
- Using the user-provided live provider on 2026-03-21 showed a separate compatibility blocker:
  `GET /models` succeeded, but both websocket and HTTPS attempts to `/v1/responses` returned `404`, so `codex exec` could not complete even a trivial probe with model `mimo-v2-pro`.
- Running the benchmark with that live provider produced a formal failure report rather than a skill judgment:
  candidate exit code `1`, no timeout, and `candidate-run-failed` because the provider did not support the Responses API path Codex expects.
- A first manual live validation using the same provider's `POST /v1/chat/completions` endpoint injected [`SKILL.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/SKILL.md), [`system/coordinator.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/system/coordinator.md), and the first benchmark scenario prompt directly.
- That first manual path did not return a clean user-facing diagnosis:
  it emitted pseudo `tool_call` blocks asking to read `.evolution` files, as if the model were still inside a toolful agent runtime.
- An ad hoc judge pass over that raw `chat/completions` output scored:
  - task classification `1/2`
  - retrieval plan `2/2`
  - capability risk diagnosis `0/2`
  - verification-first strategy `0/2`
- A second manual `chat/completions` run used a more benchmark-shaped contract:
  it injected the same skill documents, required a final user-facing answer, and explicitly stated that no tools were available.
- In that second manual path, the first benchmark scenario (`pre-task-risk-diagnosis`) produced a strongly protocol-shaped answer:
  explicit task classification, retrieval targets, capability risk diagnosis, and a verification-first execution strategy.
- An ad hoc judge pass over that second manually generated answer scored all four required criteria at `2/2`.
- After adding the `chat/completions` backend to the repository's benchmark runner, a one-scenario live run completed successfully on 2026-03-21:
  [`report.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/eval-results/model-loop-20260321T155129Z/report.md)
- That native runner report shows:
  - passed `1/1` scenarios
  - candidate exit code `0`
  - judge exit code `0`
  - score ratio `1.00`
- The generated candidate output for that run is a clean user-facing diagnosis rather than pseudo tool calls:
  [`candidate-last-message.md`](/Users/uynil/projects/agents/examples/self-evolving-agent/eval-results/model-loop-20260321T155129Z/pre-task-risk-diagnosis/candidate-last-message.md)
- The generated judge output is schema-shaped JSON and scores all four required criteria at `2/2`:
  [`judge-output.json`](/Users/uynil/projects/agents/examples/self-evolving-agent/eval-results/model-loop-20260321T155129Z/pre-task-risk-diagnosis/judge-output.json)

## Inferences

- This project is best understood as a meta-agent governance and learning layer, not as a direct research-task agent.
- The main innovation is not a novel runtime planner, but a stricter learning-state machine that tries to stop agents from confusing logging with mastery.
- The system distributes responsibility across three planes:
  - prompt/protocol plane in `SKILL.md` and modules
  - persistent memory plane in `.evolution` ledgers
  - operational integration plane in hooks, bootstrap scripts, and benchmarks
- Compared with `autoresearch`, `gpt-researcher`, and `DeepSearchAgent-Demo`, this project sits one level higher in the stack:
  it governs how an agent should learn across tasks, rather than how it should solve one research task.
- The project is unusually explicit about promotion risk:
  it treats overgeneralization as a first-class failure mode and requires transfer evidence before long-term rule persistence.
- The light-loop/full-loop split is a cost-control mechanism:
  the project tries to avoid spending full reflective overhead on every trivial task.
- The benchmark design suggests the author wants not only structural completeness, but behavioral compliance under model-in-the-loop evaluation.
- The repository's own validation story is split into two layers:
  structural compliance is already executable and passing, while behavioral benchmarking is present but not yet robust under the observed timeout path.
- The original `codex` plus Responses-API path is still incompatible with the user-provided provider, but that is no longer the only native runner path.
- There is now partial positive live evidence for the skill itself:
  when the protocol documents are present and the manual `chat/completions` path also states the runtime contract clearly, the model can match the benchmark's first scenario well.
- The manual live-validation result is prompt-sensitive:
  injecting the skill documents alone is not enough when the provider does not supply a real tool runtime, because the model may emit pseudo tool calls instead of a final answer.
- This means the integration gap is not just "Responses API vs chat completions".
  It is also "toolful local skill semantics vs tool-less compatible provider semantics".
- The `chat/completions` bridge is now encoded in the repository's native benchmark runner rather than living only in an ad hoc manual script.
- Integration confidence improved from "manual-only positive signal" to "native runner positive signal for one scenario", but coverage is still incomplete because only the first scenario has been re-run through the new backend.

## Open Questions

- In live usage, how often does the agent actually follow the full 10-step loop versus collapsing back to shallow logging habits?
- Does the capability-evolution framework materially improve future task performance, or mainly improve the quality of retrospective records?
- How much friction do the many ledgers create for day-to-day use, and does the light-loop mode reduce that enough?
- Is the benchmark suite strong enough to distinguish superficial template compliance from real capability-evolution behavior?
- Is the remaining `codex` timeout behavior mostly an aggressive timeout default, or evidence that the benchmark prompt path is too heavy for the current execution budget?
- If the provider only supports chat-completions-style compatibility and not `/responses`, should live validation use a different runner path or a different provider?
- How stable is the successful native `chat/completions` result across the other benchmark scenarios, not just the first one?
- How much of the strong manual result came from injecting only `SKILL.md` and `coordinator.md`, how much came from the explicit no-tools clause, and would adding module documents materially change quality?
- How well does the migration layer work in practice when legacy self-improving logs are messy or inconsistent?
- Which parts of the system are essential for impact:
  agenda, capability map, training units, evaluation ladder, promotion gate, or hook injection?

## Improvement Ideas

- Deep-dive the prompt architecture:
  how `SKILL.md`, `coordinator.md`, and the modules divide authority and avoid duplication.
- Analyze state management separately:
  this project appears rich enough to deserve dedicated notes on ledgers, state transitions, and resumability.
- Compare its evaluation ladder with the lighter memory-based learning systems in other repos to see what extra rigor actually buys.
- Test whether the benchmark suite catches shallow compliance by inspecting judge criteria and scenario outputs more closely.
- Re-run the remaining benchmark scenarios through the new `chat/completions` backend to see whether the first passing result generalizes.
- Inspect the remaining `codex` timeout behavior:
  the crash bug is now covered by tests, but the realistic timeout budget question remains.
- Compare the new native `chat/completions` path against the original `codex` path when a Responses-compatible provider is available.
- Compare this project against `self-improving-agent` directly if that source is added to the repo later.

## Related Notes

- [`llm-orchestration.md`](/Users/uynil/projects/agents/docs/research/projects/self-evolving-agent/llm-orchestration.md)
- [`prompts-analysis.md`](/Users/uynil/projects/agents/docs/research/projects/self-evolving-agent/prompts-analysis.md)
- [`context-management.md`](/Users/uynil/projects/agents/docs/research/projects/self-evolving-agent/context-management.md)
- [`notes.md`](/Users/uynil/projects/agents/docs/research/projects/self-evolving-agent/notes.md)
