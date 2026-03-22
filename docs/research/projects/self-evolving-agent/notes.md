---
project: self-evolving-agent
source_path: examples/self-evolving-agent
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: run the remaining benchmark scenarios through the new native `chat/completions` backend and compare them with the `codex` path when a Responses-compatible provider is available
---

# Notes

## Round: Runtime Validation

### Current Core Question

Does `self-evolving-agent` have only a well-structured protocol surface, or can its own validation tooling also demonstrate live behavioral compliance?

### Files Checked In This Round

- `examples/self-evolving-agent/scripts/run-evals.py`
- `examples/self-evolving-agent/scripts/run-benchmark.py`
- `examples/self-evolving-agent/tests/test_run_benchmark.py`
- `examples/self-evolving-agent/benchmarks/suite.json`
- `examples/self-evolving-agent/benchmarks/schemas/judge-output.schema.json`
- generated outputs under `examples/self-evolving-agent/eval-results/`

### Commands Run

- `python3 examples/self-evolving-agent/scripts/run-evals.py examples/self-evolving-agent`
- `codex --version`
- `python3 examples/self-evolving-agent/scripts/run-benchmark.py --skill-dir examples/self-evolving-agent --candidate-model gpt-5.4-mini --judge-model gpt-5.4-mini --max-scenarios 1 --timeout-seconds 90`
- `OPENAI_BASE_URL=... OPENAI_API_KEY=... codex exec --skip-git-repo-check --ephemeral -s read-only -C examples/self-evolving-agent -m mimo-v2-pro -o /tmp/self-evo-probe.txt 'Reply with exactly: OK'`
- `OPENAI_BASE_URL=... OPENAI_API_KEY=... python3 examples/self-evolving-agent/scripts/run-benchmark.py --skill-dir examples/self-evolving-agent --candidate-model mimo-v2-pro --judge-model mimo-v2-pro --max-scenarios 1 --timeout-seconds 60`
- direct HTTP probe of `GET /v1/models` and `GET /v1/responses` against the same provider
- manual `POST /v1/chat/completions` candidate probe with only a trivial prompt
- manual `POST /v1/chat/completions` scenario run with injected `SKILL.md` and `system/coordinator.md`
- manual `POST /v1/chat/completions` judge pass over that scenario output
- manual `POST /v1/chat/completions` scenario re-run with a benchmark-shaped "final answer only / no tools available" contract
- manual `POST /v1/chat/completions` judge pass over that re-run output
- `python3 -m unittest examples/self-evolving-agent/tests/test_run_benchmark.py`
- `OPENAI_API_KEY=... python3 examples/self-evolving-agent/scripts/run-benchmark.py --skill-dir examples/self-evolving-agent --backend chat-completions --chat-completions-url https://api.xiaomimimo.com/v1/chat/completions --candidate-model mimo-v2-pro --judge-model mimo-v2-pro --max-scenarios 1 --timeout-seconds 120`

### Observed Facts

- The structural compliance suite passed 12/12 checks in the current repo state.
- `codex-cli 0.116.0-alpha.10` is available in the environment.
- The benchmark suite attempted the `pre-task-risk-diagnosis` scenario first.
- The candidate prompt file was written to:
  `examples/self-evolving-agent/eval-results/model-loop-20260321T151652Z/pre-task-risk-diagnosis/candidate-prompt.txt`
- The candidate run timed out after 90 seconds before producing a completed benchmark report.
- After the timeout, the benchmark runner crashed in `run_codex_exec()` because the timeout handler tried to concatenate `stdout`/`stderr` with a string timeout suffix and hit:
  `TypeError: can't concat str to bytes`
- The crash site is in [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) around lines 113-117.
- Because of the crash, no `candidate-events.jsonl` file was written for the timed-out run.
- The user-provided provider is reachable and lists models successfully at `/v1/models`, including `mimo-v2-pro`.
- The same provider returns `404 Not Found` for `/v1/responses`.
- A trivial `codex exec` probe with that provider and model `mimo-v2-pro` failed after repeated websocket retries and an HTTPS fallback because Codex expects the Responses API path.
- Re-running the one-scenario benchmark with that provider completed without crashing and wrote a benchmark report under:
  `examples/self-evolving-agent/eval-results/model-loop-20260321T152620Z`
- That report shows:
  - candidate exit code `1`
  - candidate timed out `no`
  - pass `no`
  - blocking failure `candidate-run-failed`
- The candidate event log for that run records repeated `404` failures for both:
  - `wss://api.xiaomimimo.com/v1/responses`
  - `https://api.xiaomimimo.com/v1/responses`
- The same provider successfully answered a trivial direct `chat/completions` probe with model `mimo-v2-pro`.
- A first more faithful manual live run was then executed by injecting:
  - `SKILL.md`
  - `system/coordinator.md`
  - the first benchmark scenario prompt
- That first manual scenario run did not return a clean user-facing diagnosis.
  Instead, it emitted pseudo `tool_call` blocks asking to read `.evolution` files.
- A separate manual judge pass over that first answer returned JSON and scored:
  - task classification `1/2`
  - retrieval plan `2/2`
  - capability risk diagnosis `0/2`
  - verification-first strategy `0/2`
- A second manual `chat/completions` scenario run reused the same provider and model, but changed the prompt contract to be closer to the benchmark runner:
  - final answer only
  - no benchmark internals
  - no tools available in the environment
- That second manual scenario run returned a detailed pre-task diagnosis with:
  - explicit task classification (`unfamiliar`, `high`, `medium`)
  - retrieval targets
  - capability risk diagnosis
  - a verification-first phased execution strategy
- A separate manual judge pass over that second answer returned JSON and scored all four required criteria at `2/2`.
- The repository now contains a dedicated test file for the benchmark runner:
  [`tests/test_run_benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/tests/test_run_benchmark.py)
- Running that test file on 2026-03-21 passed `4/4` tests.
- The current tests cover:
  - timeout-path decoding for `subprocess.TimeoutExpired`
  - no-tools contract insertion in candidate prompts
  - `chat/completions` candidate message construction with injected skill context
  - explicit JSON-output instructions for chat-based judge prompts
- [`scripts/run-benchmark.py`](/Users/uynil/projects/agents/examples/self-evolving-agent/scripts/run-benchmark.py) now exposes a native `--backend chat-completions` mode plus `--chat-completions-url`.
- Running the native benchmark runner with that new backend against the same provider and model wrote a report under:
  `examples/self-evolving-agent/eval-results/model-loop-20260321T155129Z`
- That native runner report shows:
  - passed `1/1`
  - candidate exit code `0`
  - candidate timed out `no`
  - judge exit code `0`
  - judge timed out `no`
  - score ratio `1.00`
- The generated candidate output for that native run is a clean user-facing diagnosis rather than pseudo tool calls.
- The generated judge output is schema-shaped JSON and scores all four required criteria at `2/2`.

### Inferences

- The project's structural validation path is healthy enough to support further research.
- The behavioral benchmark path is currently blocked by at least one real runner bug on the timeout path.
- The timeout bug obscures the more important question of whether the benchmark scenario would have passed with a longer runtime budget.
- The current live-validation path is blocked by two independent issues:
  candidate latency plus timeout-handler robustness on one path, and provider incompatibility with Codex's Responses API on another.
- The custom provider failure is not evidence against the skill itself; it is evidence that the chosen live runner and provider protocol do not match.
- The provider is still useful for live validation if the test path uses `chat/completions` directly.
- However, direct skill-context injection is not sufficient by itself when the target provider does not supply a real tool runtime.
  The runtime contract also needs to be stated explicitly, or the model may emit pseudo tool calls.
- That prompt sensitivity has now been partially converted into repository code:
  the new native `chat/completions` backend preserves both the skill context and the no-tools runtime contract.
- The original `codex` plus `/responses` path is still incompatible with the user-provided provider, but it is no longer the only native benchmark path.
- Validation confidence is higher than before because the positive result is now reproduced through the repository's own benchmark runner, not only through an ad hoc manual script.
- Coverage is still incomplete because only the first scenario has been re-run through the new backend.

### Current Hypothesis

The next useful step is no longer deciding whether to add a `chat/completions` path.
That path now exists and has passed one real scenario.
The next narrower question is whether the remaining benchmark scenarios also hold up through the same native backend.

### Blockers

- External blocker:
  the user-provided provider exposes `/v1/models` but not `/v1/responses`, which prevents `codex exec` from using it successfully.
- Coverage blocker:
  only the first scenario has been re-validated through the new native `chat/completions` backend.
- Comparison blocker:
  the original `codex` path still cannot be compared against the same provider because the provider does not expose `/responses`.

### Next Round Question

Do the positive results hold up across the remaining benchmark scenarios now that the `chat/completions` bridge lives inside the repository's normal benchmark runner?

### Resumable Context Summary

The project already has good static evidence for a layered capability-evolution protocol.
Runtime validation now has two separate findings:
`run-evals.py` passed, one benchmark path exposed a local timeout bug, and the user-provided live provider was confirmed reachable at `/models` but incompatible with Codex because `/responses` returns `404`.
There are now two manual `chat/completions` results:
the first injected-skill run leaked pseudo tool calls and failed most required criteria, while the second benchmark-shaped run added a no-tools contract and then passed the first scenario with `2/2` on all required criteria.
There is now a third and stronger result:
after patching the native benchmark runner with a `chat/completions` backend, the first scenario passed through the repository's own report flow with `1.00` score ratio and schema-shaped judge output.
Resume from the live-validation path, not from the prompt stack.
