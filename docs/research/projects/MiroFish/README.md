---
project: MiroFish
source_path: examples/MiroFish
status: partially-validated
confidence: medium
last_updated: 2026-03-22
next_action: run one full report generation against a warm simulation and inspect how ReportAgent mixes graph retrieval, live interviews, and fallback paths
---

# MiroFish

This is the canonical entry page for the deeper second-pass research notes on `examples/MiroFish`.

## Observed Facts

- The project is a Flask + Vue application with a Zep knowledge graph, OASIS social simulation, and a report-generation layer.
- The backend creates a project, extracts text from uploaded documents, generates an ontology, builds a Zep graph, prepares simulation profiles/config, runs Twitter and Reddit simulations, then generates a report.
- `ProjectManager`, `SimulationManager`, `SimulationRunner`, and `ReportManager` all persist artifacts to the filesystem instead of relying on a database.
- The report layer is not just a static summarizer. It uses multiple tools, structured logs, per-section ReAct loops, and a separate post-report chat surface.
- `ReportAgent` is internally split into three modes:
  - graph-grounded outline planning
  - section-scoped ReAct report writing
  - lighter post-report chat
- `SimulationRunner` can optionally batch meaningful agent actions back into Zep graph memory while the simulation is still running.
- The simulation runtime keeps per-platform `actions.jsonl` files, `run_state.json`, IPC command/response folders, and supports interview commands against the live OASIS environment.
- `TaskManager` is only in-memory, while project, simulation, and report artifacts are durable on disk.
- Runtime validation is now grounded in a real credentialed pass:
  - backend source compiles locally
  - `run.py` reaches config validation cleanly in an isolated venv
  - `create_app()` succeeds
  - ontology generation succeeds through the Flask API
  - graph build completes and returns real Zep graph data
  - simulation creation and prepare both complete and write artifact files
- The project reads generic OpenAI-style variables (`LLM_API_KEY`, `LLM_BASE_URL`, `LLM_MODEL_NAME`) rather than provider-specific names, so any compatible external provider must be mapped into those fields explicitly.
- The provided Kimi coding configuration maps into those fields correctly, but a direct request to `kimi-for-coding` via the backend's OpenAI-style client currently returns HTTP 403 and says the model is only available to supported coding-agent clients.
- A second OpenAI-compatible configuration was validated successfully through MiroFish's own `LLMClient`, including both normal chat and JSON-mode output.
- With working LLM and Zep credentials, one live graph build completed end-to-end and returned a small real graph from Zep, and one simulation prepare run completed with 5 generated entity profiles and persisted config/state artifacts.
- A subsequent simulation start request launched the background runner successfully, and `simulation.log` shows the Twitter script reached model init, profile loading, OASIS env creation, initial-post publishing, simulation-loop completion, and wait-for-command mode.
- The apparent "stuck at round 0" signal comes from an observability mismatch: `SimulationRunner` monitors `twitter/actions.jsonl`, but the single-platform `run_twitter_simulation.py` path does not appear to emit those action logs.
- The later `SIGTERM` and `服务器关闭，模拟被终止` state were likely induced by the one-off Flask probe process exiting, because `create_app()` registers cleanup logic that terminates child simulations on server shutdown.
- Under a persistent backend process, the same simulation can be restarted and `env-status` turns `env_alive=true`, which confirms that the wait-command mode survives correctly when the parent server stays up.
- Under that persistent setup, `env_alive=true` does not appear immediately after `/start`; in the validated run, the single-platform path needed roughly 18-20 seconds of warm-up before the wait-command loop was actually ready to accept IPC.
- Once `env_alive=true` is reached, single-platform `/api/simulation/interview` works for both short and richer prompts, interview records are written into `twitter_simulation.db`, and `/api/simulation/close-env` exits the environment cleanly.
- `env-status` still reports `twitter_available=false` even while the environment is alive, because the single-platform `env_status.json` shape only records `status` and `timestamp`, while the backend expects richer per-platform fields.
- The provided `test_profile_format.py` script runs, but its own output reports a mismatch between the generated profile formats and the script's expected OASIS reference schema.
- `/api/report/generate` starts report generation in a background thread after checking the simulation/project/graph metadata, but it does not wait for the live simulation to become interview-ready.
- Inside report generation, `interview_agents` is therefore a best-effort evidence source rather than a pre-guaranteed capability.

## Inferences

- The core product shape is a staged prediction pipeline, not a single chatbot and not a single monolithic agent runtime.
- The strongest LLM-driven behavior lives in the report layer, where the model decomposes evidence gathering across graph search and live interviews.
- The system is really two layers stacked together:
  - a durable graph-to-simulation production pipeline
  - a bounded research/report layer on top of that pipeline
- Context is intentionally bounded at every stage, which suggests the design optimizes for long artifacts without letting prompts grow unbounded.
- The filesystem-first design makes long jobs inspectable and recoverable, but the in-memory task manager means some progress metadata may be lost across backend restarts.
- The graph-construction and simulation-prepare layers are clearly live-validated, and the simulation-start path is more functional than it first looked.
- The earlier IPC timeout looked like a command-loop failure, but the more reproducible picture is different: single-platform IPC works after the environment reaches its true ready state.
- The remaining runtime weakness is observability and readiness signaling: start returns early, `env-status` under-reports platform availability, and run-state/action progress still does not reflect what the simulation actually did.
- `ReportAgent` should be treated as the actual agentic core of MiroFish, but its most distinctive tool (`interview_agents`) still depends on runtime readiness that the outer report API does not enforce.

## Open Questions

- Why does `SimulationRunner` rely on `twitter/actions.jsonl` for state progression when the single-platform Twitter script appears to use DB/log outputs without wiring in `action_logger`?
- Should single-platform runs produce `actions.jsonl` parity with `run_parallel_simulation.py`, or should the backend expose state from `simulation.log` / SQLite instead?
- What is the correct readiness contract after `/start` for single-platform runs: process launched, simulation loop finished, or wait-command ready?
- Should the backend block `/interview` until single-platform `env_status.json` reflects a richer ready state, instead of relying on callers to poll manually?
- What happens in a full report run if a section chooses `interview_agents` before the simulation is interview-ready: does the report degrade gracefully, stall, or overfit to graph-only evidence?
- How often does the live simulation actually need graph-memory updates in practice, versus report-time retrieval alone?
- What breaks first if the backend restarts mid-graph-build, mid-simulation, or mid-report-generation?
- Are there any additional prompt or workflow branches in the OASIS runtime scripts that only show up when the runner is kept alive under a persistent server instead of a one-shot probe?
- Does the team intend this to stay report-centric, or evolve into a broader research assistant with persistent multi-turn planning?

## Next Action

- Treat `MiroFish` as partially live-validated rather than credential-blocked.
- The persistent-server validation is now done: wait-command mode survives, interview works, and close-env works when IPC is sent after the environment is actually ready.
- The next focused pass should be a full `ReportAgent` run against a warm simulation, so the section-level tool mix and failure-handling paths can be observed in `agent_log.jsonl`.
- After that, align `env_status.json`, `run_state.json`, and single-platform action-log behavior with backend expectations so readiness and progress are observable without manual timing guesses.

## Research Files

- [architecture.md](./architecture.md)
- [llm-orchestration.md](./llm-orchestration.md)
- [prompts-analysis.md](./prompts-analysis.md)
- [context-management.md](./context-management.md)
- [deep-research-mode.md](./deep-research-mode.md)
- [notes.md](./notes.md)
