---
project: MiroFish
source_path: examples/MiroFish
status: partially-validated
confidence: medium
last_updated: 2026-03-22
next_action: run one full report generation against a warm simulation and inspect how ReportAgent behaves when graph retrieval and live interviews are both available
---

# Notes

## Observed Facts

- The repository already contains a user-facing README in Chinese, but the research docs need to stay grounded in source files.
- The backend scripts clearly define a post-simulation waiting state for interview commands.
- The report layer is the most behaviorally rich area of the codebase.
- `ReportAgent` has two distinct user-facing surfaces:
  - asynchronous report generation
  - synchronous post-report chat
- The system saves a surprisingly rich artifact trail: project files, extracted text, simulation configs, run-state, action logs, report sections, and both structured and console-style report logs.
- The async task layer is weaker than the artifact layer because `TaskManager` is memory-only.
- `python3 -m compileall backend/app` succeeds, so the backend source tree is at least syntactically loadable.
- In the default local environment, `flask_cors`, `openai`, `python-dotenv`, and `zep_cloud` are missing, so live validation cannot start from the host Python directly.
- In a temporary isolated venv with the backend requirements installed, `python run.py` exits with configuration validation errors for `LLM_API_KEY` and `ZEP_API_KEY` when no `.env` is present.
- In that isolated venv, `create_app()` succeeds with dummy `LLM_API_KEY` and `ZEP_API_KEY`, which shows the Flask app factory and blueprint registration do not immediately require live API access.
- `backend/scripts/test_profile_format.py` runs in the isolated venv, but its own validation output reports that the generated Twitter CSV and Reddit JSON do not match the script's expected OASIS reference shape.
- The backend only reads `LLM_API_KEY`, `LLM_BASE_URL`, and `LLM_MODEL_NAME`; provider-specific variable names such as `ANTHROPIC_API_KEY` are not consumed unless mapped into those generic LLM variables.
- If `LLM_MODEL_NAME` is not set, the project defaults to `gpt-4o-mini`, so any non-OpenAI compatible-provider setup still needs an explicit model override.
- A direct OpenAI-compatible test request against `https://api.kimi.com/coding/v1` with `model=kimi-for-coding` returns HTTP 403 with an error saying the model is currently only available to coding agents such as Kimi CLI, Claude Code, Roo Code, and similar tools.
- With the same Kimi-style `LLM_API_KEY`, `LLM_BASE_URL`, and `LLM_MODEL_NAME`, `create_app()` still succeeds locally, so the incompatibility shows up at actual LLM call time rather than during config parsing or app boot.
- A second OpenAI-compatible provider setup was validated through `app.utils.llm_client.LLMClient`: plain chat works and `chat_json()` also works, which means the provider supports the JSON-mode path MiroFish relies on.
- With a working OpenAI-compatible LLM setup plus real Zep access, ontology generation, graph build, simulation creation, and simulation prepare all complete successfully through the Flask API.
- One validated graph build returned real graph data from Zep rather than mock data.
- One validated simulation prepare pass wrote `state.json`, `simulation_config.json`, `reddit_profiles.json`, and `twitter_profiles.csv` into the simulation artifact directory.
- A simulation start request launches the background runner and records a live PID.
- The resulting `simulation.log` shows the Twitter runner actually progresses through:
  - model initialization
  - profile loading
  - OASIS environment creation
  - 5 initial posts
  - simulation-loop completion
  - wait-for-command mode
- The generated SQLite database contains the expected sign-up and initial-post records, so the script did real work even though `run_state.json` never advanced beyond round 0.
- The single-platform Twitter script does not appear to wire in the shared `action_logger` path that writes `twitter/actions.jsonl`, while `SimulationRunner` depends on that file for round/action visibility.
- The final `服务器关闭，模拟被终止` state lines up with the backend cleanup path that terminates child simulations when the hosting Flask process exits.
- Under a persistent Flask backend process, restarting the same simulation yields `env_alive=true`, so the wait-command environment can stay up when the parent process is not torn down.
- In that persistent setup, `env_alive=true` does not appear immediately; there is a real warm-up window after `/start` before the environment is ready for IPC, and in the validated run that delay was roughly 18-20 seconds.
- After waiting for the environment to report `env_alive=true`, single-platform `/api/simulation/interview` succeeds for both a short prompt and a richer analytical prompt.
- In the validated run, a short interview returned in roughly 13 seconds and a richer analytical interview returned in roughly 20 seconds.
- The successful interviews are persisted and show up through `/api/simulation/interview/history`, which confirms that `ManualAction.INTERVIEW` is operational in the validated run.
- `/api/simulation/close-env` also succeeds cleanly once the environment is actually ready.
- The single-platform `env_status.json` never exposes `twitter_available` / `reddit_available`, so `env-status` under-reports platform availability even when the environment is alive.
- `/api/report/generate` launches a background thread after checking simulation/project/graph metadata, but it does not wait for interview readiness.
- `ReportAgent.generate_report()` persists a rolling artifact trail:
  - `meta.json`
  - `outline.json`
  - `progress.json`
  - `section_XX.md`
  - `full_report.md`
  - `agent_log.jsonl`
  - `console_log.txt`
- `ReportAgent` is internally hierarchical:
  - `plan_outline()` plans a short report from graph context
  - `_generate_section_react()` runs bounded section-level research
  - `chat()` reuses the saved report as compressed follow-up context
- The section loop carries forward previous sections, but only as truncated prose (4000 chars per completed section).
- The chat surface truncates saved report content to 15000 characters and only keeps the last 10 chat turns.
- `interview_agents` is not a single primitive call; it:
  - loads profiles from persisted simulation artifacts
  - uses LLM to select interview targets
  - uses LLM to generate interview questions
  - calls `SimulationRunner.interview_agents_batch(...)`
  - cleans response wrappers
  - extracts quotes
  - uses LLM again to summarize the interviews

## Runtime Validation Snapshot

- Verified locally without network credentials:
  - backend source compiles
  - app import path works after dependencies are installed
  - `run.py` reaches config validation and fails cleanly on missing keys
  - `create_app()` works with dummy keys
  - `test_profile_format.py` runs and exposes a format mismatch
- Verified with the provided Kimi coding endpoint:
  - the config variable mapping is correct
  - app boot works with the mapped values
  - direct LLM calls to `kimi-for-coding` are rejected with HTTP 403
- Verified with a second OpenAI-compatible provider:
  - `LLMClient.chat()` returns a normal response
  - `LLMClient.chat_json()` returns a parsed JSON object successfully
  - `run.py` then stops only at missing `ZEP_API_KEY`
- Verified with real working LLM and Zep credentials:
  - `/api/graph/ontology/generate` succeeds
  - `/api/graph/build` completes asynchronously and returns a real graph from Zep
  - `/api/simulation/create` succeeds
  - `/api/simulation/prepare` reaches `status: ready`
  - prepare output includes 5 generated entity profiles and persisted simulation config/state files
- Verified but still unresolved:
  - `/api/simulation/start` launches the runner successfully
  - `simulation.log` shows the runner reaches wait-for-command mode rather than hanging in early initialization
  - a persistent backend process keeps the environment alive correctly
  - `/api/simulation/interview` works after the environment becomes truly ready
  - `/api/simulation/close-env` works and cleanly shuts the environment down
  - `run_state.json` still stays at round 0 because the backend monitor watches `twitter/actions.jsonl`
  - `twitter/actions.jsonl` is still not produced by the single-platform script path in this validation
  - `env_status.json` is still too sparse for the richer backend status view
  - the earlier probe-style validation process still explains the previous `SIGTERM` teardown, and earlier IPC timeouts now look more like readiness/timing artifacts than a stable command-loop failure
  - no full report generation run has yet been validated end-to-end under a warm live simulation, so actual `ReportAgent` tool-choice behavior is still inferred from source rather than observed in logs

## Current Runtime Gap

- The project is no longer blocked on missing credentials.
- The current blocker is not basic startup or basic IPC.
- The real remaining gap is that single-platform readiness and progress are poorly surfaced:
  - the backend monitor expects logs/status fields the single-platform script does not emit
  - callers need to discover readiness by polling, because `/start` returns before the environment is truly interview-ready
- There is now a second, more report-specific gap:
  - `ReportAgent` can ask for live interviews
  - the outer report API does not guarantee those interviews are ready at report start time
  - so the richest evidence path is available only on a best-effort basis

## Concrete Alignment Targets

- `app/api/simulation.py` returns success as soon as `SimulationRunner.start_simulation()` launches the subprocess, so `/start` currently signals "process started" rather than "IPC ready".
- `app/services/simulation_runner.py` monitors `twitter/actions.jsonl` / `reddit/actions.jsonl` to advance `run_state.json`, but single-platform scripts do not emit those files.
- `scripts/run_twitter_simulation.py` and `scripts/run_reddit_simulation.py` write `env_status.json` with only `status` and `timestamp`, while the parallel path also writes `twitter_available` / `reddit_available`.
- `scripts/run_parallel_simulation.py` logs round start/end plus action records into `actions.jsonl`; single-platform mode currently relies on SQLite and console log output instead of the same structured progress channel.
- The practical result is a three-layer contract mismatch:
  - API layer says `running`
  - env-status eventually says `alive`
  - run-state still says round 0 / no actions
- `app/api/report.py:/generate` assumes graph-ready and simulation-linked state are enough to start report generation, but it does not distinguish:
  - simulation exists
  - simulation process launched
  - simulation interview-ready
- `app/services/report_agent.py` assumes `interview_agents` is just another optional tool inside the section loop, so readiness failure is currently handled at tool-call time rather than by outer orchestration.

## Inferences

- The best next research pass is likely a live run, not more static reading.
- If there is a single area most worth stress-testing, it is the boundary between the report agent and the live OASIS interview path.
- MiroFish is more interesting as a prediction pipeline with an embedded research agent than as a generic multi-agent framework.
- The deepest architectural insight is that report generation is not downstream decoration; it is where graph memory and live simulation finally get fused into one reasoning loop.
- The startup path is well layered enough that dependency issues and configuration issues can be separated cleanly; once dependencies are present, the next real blocker is credentials, not app wiring.
- The next blocker is not only missing Zep credentials; the chosen Kimi coding model also appears incompatible with this backend's plain OpenAI-style call path.
- Once a callable OpenAI-compatible LLM and real Zep access are present, the graph and prepare phases work; the unresolved risk shifts to early-stage simulation execution rather than setup.
- The earlier "simulation start stall" diagnosis was too coarse. The deeper issue is observability drift between the backend monitor and the single-platform script, not a confirmed deadlock in OASIS startup.
- The persistent-server validation removes another uncertainty: the environment can stay alive, `interview` can work, and `close_env` can work, so the remaining fault is largely about readiness signaling and observability rather than broken IPC semantics.
- The profile-format script currently behaves more like a compatibility probe than a passing regression test, because it demonstrates a mismatch instead of confirming conformance.
- The simulation stack likely works better under a long-lived backend than under a one-shot Flask test probe, because the app factory registers cleanup that kills child runners on shutdown.
- `ReportAgent` is the actual bounded deep-research core of MiroFish, not just a report formatter.
- The report loop mixes two evidence classes with different readiness guarantees:
  - graph retrieval, which is available once graph build succeeds
  - live simulation interviews, which depend on runtime readiness and warm-up
- The current system is better at recovering from evidence-format errors than from evidence-availability timing issues.

## Open Questions

- Do all live simulations end in a wait-for-command state by default, or only the parallel runner?
- Is the report agent expected to be the primary user-facing assistant after report generation, or just one of several interaction surfaces?
- Does the team want deeper graph-memory updates during simulation, or is report-time retrieval the intended main path?
- If the backend restarts during graph build, simulation, or report generation, which exact pieces recover automatically and which must be reconstructed manually?
- Is the profile export shape intentionally different from the script's "expected OASIS format" examples, or is the script capturing a real integration drift?
- Is the missing `twitter/actions.jsonl` support in single-platform mode intentional, unfinished, or a regression relative to `run_parallel_simulation.py`?
- What objective signal should the backend expose for “IPC ready” in single-platform mode, beyond the coarse `env_alive` file?
- Should the single-platform scripts reuse the shared IPC server abstraction and richer status schema from the parallel path to reduce drift?
- In a real report run, how often does the section loop choose `interview_agents`?
- If a section chooses `interview_agents` too early, does the report recover by switching tools, or does it simply absorb the failure text as weak evidence?
- Does the 3-tool minimum create materially better sections, or mainly increase cost and latency for short sections?

## Improvement Ideas

- Add a small schema note for `actions.jsonl`, `run_state.json`, and report artifacts if the next pass confirms they are stable.
- Add a short evidence matrix if the report workflow starts needing auditable citations per section.
- Consider documenting the exact live interview result shape once a real run verifies it.
- If production resilience matters, consider persisting task progress the same way projects, simulations, and reports are already persisted.
- If the profile-format script is intended as a guardrail, convert its printed mismatches into a real failing test with an explicit expected schema decision.
- Bring `run_twitter_simulation.py` and `run_reddit_simulation.py` into parity with `run_parallel_simulation.py` for action-log emission, or teach `SimulationRunner` to derive progress from the single-platform DB/log outputs.
- Add a dedicated long-lived validation recipe for IPC/wait-command testing so research probes do not accidentally kill the child process on exit.
- Align single-platform `env_status.json` with the richer shape expected by `SimulationRunner.get_env_status_detail()`.
- Consider replacing the ad-hoc single-platform IPC handler with the shared `simulation_ipc.py` contract to eliminate drift.
- Add an explicit readiness marker for “wait-command loop entered and first IPC poll completed” so callers do not have to infer readiness from timing.
- Add a report-generation preflight that distinguishes graph-ready from interview-ready, or expose tool availability to `ReportAgent` so it can choose graph-only paths intentionally when live interviews are not yet ready.
