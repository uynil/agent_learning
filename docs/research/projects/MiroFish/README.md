---
project: MiroFish
source_path: examples/MiroFish
status: partially-validated
confidence: medium
last_updated: 2026-03-22
next_action: verify and compare recovery policies for multiple reports under one `simulation_id`, because restart-time lookup currently diverges between direct `report_id` access and by-simulation access
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
- The prompt stack is layered rather than monolithic:
  - planning prompt
  - section prompt
  - chat prompt
  - loop-time repair/control messages injected by Python
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
- One full live report run against a warm simulation now completes end-to-end:
  - report id `report_7e34e64b29aa`
  - 3 generated sections plus `full_report.md`
  - total logged runtime about 316 seconds
- In that validated run, all 3 sections used the same core tool pattern:
  - `panorama_search`
  - `insight_forge`
  - `interview_agents`
- Section 2 added a fourth tool call to `quick_search` after the interview result came back.
- The live interview path succeeded structurally in all 3 sections:
  - batch IPC returned `completed`
  - 4 simulated agents were selected and processed
  - interview summaries were written into `agent_log.jsonl`
- The report-layer interview summaries still showed empty per-agent answers in all 3 sections, with both Twitter and Reddit blocks rendered as `未获得回复`.
- Follow-up inspection of the underlying IPC response files shows that the single-platform runner actually returned substantive interview text for those same batch requests.
- The immediate cause is a response-schema mismatch:
  - single-platform `run_twitter_simulation.py` returns batch results keyed by bare agent IDs such as `"0"` and `"3"`
  - `zep_tools.py` always parses batch results as if they were dual-platform keys such as `twitter_0` and `reddit_0`
  - the report tool therefore drops the real single-platform answers and falls back to the empty placeholders
- The report tool did load `reddit_profiles.json` in the validated Twitter-only run, but spot checks show the selected indices and usernames still line up with `twitter_profiles.csv`, so that profile-source preference looks like a secondary coupling risk rather than the main cause of empty interview evidence.
- Even with those empty interview results, all 3 sections still completed and the final report was assembled successfully.
- `agent_log.jsonl` captured section-level tool sequencing and progress accurately during the live run, while `progress.json` stayed stale during long interview waits and only caught up at completion.
- The section loop's actual working state is a compound of:
  - `messages`
  - `tool_calls_count`
  - `used_tools`
  - `conflict_retries`
  - truncated `previous_sections`
  - tiny `report_context`
- Session management is split across four layers:
  - simulation runtime session
  - report generation session
  - in-memory task session
  - request-scoped chat session
- The restart semantics are asymmetric:
  - `simulation_id` and `report_id` can be reloaded from disk-backed artifacts
  - `task_id` cannot be recovered after backend restart
  - chat can only be reconstructed from saved report artifacts plus client-provided history
- A real mid-report restart probe now confirms that asymmetry empirically:
  - the interrupted `report_id` remained inspectable after restart
  - the old `task_id` was immediately dead
  - chat reconstruction by `simulation_id` did not reliably pick the interrupted report

## Inferences

- The core product shape is a staged prediction pipeline, not a single chatbot and not a single monolithic agent runtime.
- The strongest LLM-driven behavior lives in the report layer, where the model decomposes evidence gathering across graph search and live interviews.
- The system is really two layers stacked together:
  - a durable graph-to-simulation production pipeline
  - a bounded research/report layer on top of that pipeline
- Context is intentionally bounded at every stage, which suggests the design optimizes for long artifacts without letting prompts grow unbounded.
- The filesystem-first design makes long jobs inspectable and recoverable, but the in-memory task manager means some progress metadata may be lost across backend restarts.
- The project is better at recovering outputs than recovering control flow:
  - saved simulation/report artifacts are durable
  - active report loops, task polling state, and live in-memory loop variables are not
- Recovery is also handle-sensitive:
  - direct `report_id` retrieval is strong
  - `simulation_id` lookup is currently ambiguous when multiple reports exist
- That ambiguity is now traced to a concrete policy split:
  - `ReportManager.get_report_by_simulation()` returns the first matching folder from `os.listdir()`
  - simulation history uses a different "latest created report" policy
- The graph-construction and simulation-prepare layers are clearly live-validated, and the simulation-start path is more functional than it first looked.
- The earlier IPC timeout looked like a command-loop failure, but the more reproducible picture is different: single-platform IPC works after the environment reaches its true ready state.
- The remaining runtime weakness is observability and readiness signaling: start returns early, `env-status` under-reports platform availability, and run-state/action progress still does not reflect what the simulation actually did.
- `ReportAgent` should be treated as the actual agentic core of MiroFish, but its most distinctive tool (`interview_agents`) still depends on runtime readiness that the outer report API does not enforce.
- The report layer is now validated as more than “theoretically agentic”: it really does run sequential section-level research loops against a live simulation.
- The prompt system is implementation-coupled rather than prompt-only:
  - prompt templates define the behavioral contract
  - Python enforces budgets, repair paths, truncation, and closure
- Context management is intentionally lossy and stage-specific:
  - planning only sees a small graph snapshot
  - section writing only sees truncated prior sections
  - chat only sees truncated report markdown plus recent turns
- In practice, the current report loop appeared graph-dominant in the validated run, but that result is now partly explained by an integration bug rather than by intrinsically weak live interviews.
- The 3-tool minimum is not just a design intention; it is live behavior. In this run it pushed every section through the same minimum research ritual, even when the interview branch produced no usable content.
- The report-layer “no usable interview content” outcome is now better understood as a tool-adapter bug, not proof that the simulation environment failed to generate answers.
- `agent_log.jsonl` is the most trustworthy runtime truth source for report execution. `progress.json` is a user-facing coarse indicator, not a reliable execution trace during long tool calls.

## Open Questions

- Why does `SimulationRunner` rely on `twitter/actions.jsonl` for state progression when the single-platform Twitter script appears to use DB/log outputs without wiring in `action_logger`?
- Should single-platform runs produce `actions.jsonl` parity with `run_parallel_simulation.py`, or should the backend expose state from `simulation.log` / SQLite instead?
- What is the correct readiness contract after `/start` for single-platform runs: process launched, simulation loop finished, or wait-command ready?
- Should the backend block `/interview` until single-platform `env_status.json` reflects a richer ready state, instead of relying on callers to poll manually?
- What happens in a full report run if a section chooses `interview_agents` before the simulation is interview-ready: does the report degrade gracefully, stall, or overfit to graph-only evidence?
- Should the single-platform batch interview response be normalized to the same `twitter_{id}` / `reddit_{id}` schema as the parallel runner, or should `zep_tools.py` accept both schemas?
- The report tool loading `reddit_profiles.json` in a Twitter-only run appears to be more of a coupling smell than the primary failure, because the validated simulation kept the same agent ordering and usernames across Reddit and Twitter profile exports.
- How often does the live simulation actually need graph-memory updates in practice, versus report-time retrieval alone?
- What breaks first if the backend restarts mid-graph-build, mid-simulation, or mid-report-generation?
- If the backend restarts mid-report, which prompt/loop/session layers can be reconstructed from artifacts and which are lost permanently?
- After a backend restart, how should the UI distinguish between:
  - an unfinished but inspectable report
  - a dead task id
  - a report that truly needs regeneration
- Which report should count as canonical for one `simulation_id` after restarts and retries:
  - latest created
  - latest completed
  - latest with markdown content
  - or a status-priority selection rule
- Should the product expose both concepts explicitly:
  - latest attempt
  - latest successful report
- Are there any additional prompt or workflow branches in the OASIS runtime scripts that only show up when the runner is kept alive under a persistent server instead of a one-shot probe?
- Does the team intend this to stay report-centric, or evolve into a broader research assistant with persistent multi-turn planning?

## Next Action

- Treat `MiroFish` as partially live-validated with one successful end-to-end report run, not merely with isolated graph/simulation probes.
- The persistent-server validation is now done: wait-command mode survives, interview works, close-env works, and full report generation also completes.
- The next focused pass should explain and, if needed, fix the remaining integration gap:
  - single-platform batch interview returns real answers
  - `zep_tools.py` currently fails to read them because it assumes a dual-platform response schema
  - the research question is now whether this mismatch is isolated or whether other report-time tool adapters make similar assumptions
- After that, verify which parts of the system are actually resumable:
  - simulation runtime
  - report generation
  - task polling
  - post-report chat
- The current code reading implies this provisional matrix:
  - simulation artifacts: reloadable
  - report artifacts: reloadable
  - task polling state: not reloadable
  - chat continuity: reconstructable but not resumable
- The new probe adds one more line:
  - by-simulation report recovery: reloadable in principle, but currently selection-unstable
- The strongest current design mismatch is therefore not artifact persistence but report selection policy consistency.

## Research Files

- [architecture.md](./architecture.md)
- [llm-orchestration.md](./llm-orchestration.md)
- [prompts-analysis.md](./prompts-analysis.md)
- [context-management.md](./context-management.md)
- [deep-research-mode.md](./deep-research-mode.md)
- [session-management.md](./session-management.md)
- [notes.md](./notes.md)
