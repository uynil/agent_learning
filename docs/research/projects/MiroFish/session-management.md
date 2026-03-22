---
project: MiroFish
source_path: examples/MiroFish
status: partially-validated
confidence: medium
last_updated: 2026-03-22
next_action: verify whether report lookup should prefer newest, completed, or markdown-bearing artifacts when multiple reports share one `simulation_id`
---

# Session Management

## Observed Facts

- `MiroFish` does not expose one unified session abstraction. It coordinates work through four different session/state layers:
  - simulation runtime session
  - report generation session
  - task-tracking session
  - chat/request session
- The main identity keys are:
  - `project_id`
  - `simulation_id`
  - `report_id`
  - `task_id`
- Simulation sessions are anchored by `simulation_id` and span:
  - `uploads/simulations/<simulation_id>/`
  - child process lifecycle
  - IPC command/response folders
  - platform SQLite DBs
  - `env_status.json`
  - optional `run_state.json`
- Report sessions are anchored by `report_id` and span:
  - a daemon thread started by `/api/report/generate`
  - `uploads/reports/<report_id>/`
  - `meta.json`
  - `outline.json`
  - `progress.json`
  - `section_XX.md`
  - `full_report.md`
  - `agent_log.jsonl`
  - `console_log.txt`
- Task sessions are anchored by `task_id`, but `TaskManager` is only an in-memory singleton and does not persist tasks to disk.
- Chat sessions are request-scoped rather than server-owned:
  - `ReportAgent.chat()` receives one message plus optional `chat_history`
  - it rebuilds context from the saved report and only the last 10 client-supplied turns
  - there is no persistent backend chat-session ID
- `create_app()` registers `SimulationRunner.register_cleanup()`, so backend shutdown attempts to terminate live simulation subprocesses.
- `ReportManager.get_report_by_simulation(simulation_id)` reconstructs report state by scanning report folders on disk rather than by querying a central DB.
- `ReportManager.get_report_by_simulation(simulation_id)` returns the first matching report encountered during `os.listdir(REPORTS_DIR)`, not the newest or healthiest report.
- The simulation-history path uses a different rule: `_get_report_id_for_simulation(simulation_id)` sorts matches by `created_at` descending and returns the newest one.
- A real mid-report restart probe was run against `simulation_id=sim_13fca91e3b2b` and a new report `report_6001abd856e2`.
- Before restart, the probe confirmed:
  - `task_id` polling worked
  - `progress.json` showed section-1 generation
  - the report folder already contained `meta.json`, `outline.json`, `progress.json`, `agent_log.jsonl`, and `console_log.txt`
- After backend restart, the old `task_id` returned `任务不存在`, but `GET /api/report/report_6001abd856e2` could still reconstruct the interrupted report from disk.
- That reconstructed interrupted report kept:
  - `status=planning`
  - saved outline data
  - empty markdown content
  - a stale `progress.json` that still pointed at section 1 generation
- Post-restart lookup by `simulation_id` did not consistently return the interrupted report:
  - `/api/report/by-simulation/<simulation_id>` returned an older failed report
  - `/api/report/check/<simulation_id>` also reported that older failed report
  - `/api/report/chat` behaved as if no usable report existed
- A local replay of the current selection rules against the same artifact set makes the divergence explicit:
  - first-match policy -> `report_92776ac4feb3` (`failed`)
  - latest-created policy -> `report_6001abd856e2` (`planning`)
  - latest-completed / markdown-bearing policy -> `report_7e34e64b29aa` (`completed`)

## Session Types

## Simulation Session

- Entry path: create -> prepare -> start -> wait-command -> interview/close.
- Durable state:
  - config/profile artifacts
  - DB files
  - IPC response files
  - logs
- Ephemeral state:
  - child process liveness
  - true readiness before `env_alive=true`
- Main weakness:
  - start/ready/progress are modeled by different artifacts that can drift.

## Report Session

- Entry path: `/api/report/generate`.
- Durable state:
  - report folder and saved sections
  - structured logs
  - final markdown
- Ephemeral state:
  - daemon thread
  - active loop variables in `_generate_section_react()`
- Main weakness:
  - a backend restart can kill the coordinator thread even while partial report artifacts already exist.

## Task Session

- Entry path: async API task creation.
- Durable state:
  - none
- Ephemeral state:
  - progress
  - status
  - result/error
- Main weakness:
  - restart loses task polling state even when underlying work products survive.

## Chat Session

- Entry path: request-time follow-up chat.
- Durable state:
  - saved report markdown
- Client-owned state:
  - `chat_history`
- Ephemeral state:
  - current tool loop
- Main weakness:
  - there is no server-owned way to resume the exact analytical conversation.

## Restart Semantics Matrix

- Simulation session:
  - survives restart at the artifact layer because `SimulationManager.get_simulation()` reloads `state.json`
  - may not survive at the live-runtime layer because subprocess liveness is separate from the saved state
  - strongest post-restart anchors:
    - `state.json`
    - `simulation_config.json`
    - profile exports
    - platform DBs
    - IPC response files
- Report session:
  - survives restart at the artifact layer because `ReportManager.get_report()` and `get_report_by_simulation()` rebuild state by scanning report folders on disk
  - does not survive as an active generation session because `/api/report/generate` uses a daemon thread and there is no persisted loop checkpoint
  - strongest post-restart anchors:
    - `meta.json`
    - `outline.json`
    - completed `section_XX.md`
    - `full_report.md`
    - `agent_log.jsonl`
- Task session:
  - does not survive restart because `TaskManager` keeps `_tasks` only in memory
  - `task_id` is therefore a transient polling handle, not a durable session key
- Chat session:
  - survives only in reconstructive form
  - after restart, the backend can still answer chat requests if:
    - `simulation_id` still resolves
    - a saved report still exists
    - the client resubmits enough `chat_history`
  - it cannot resume the exact prior tool loop because there is no server-owned chat checkpoint

## Canonical Handles After Restart

- `project_id` remains the canonical anchor for:
  - uploaded files
  - extracted text
  - ontology
  - graph id
  - simulation requirement
- `simulation_id` remains the canonical anchor for:
  - simulation artifact folders
  - profile/config/state reload
  - report lookup by simulation, but only weakly because multiple competing report-selection policies already exist
- `report_id` remains the canonical anchor for:
  - report artifacts
  - progress/section/log inspection
- `task_id` is not canonical after restart.
- There is no canonical backend chat-session handle.

## API Recovery Semantics

- `/api/report/generate` returns both `report_id` and `task_id`, but they have different durability:
  - `report_id` points to disk-backed artifacts
  - `task_id` points to in-memory polling state
- `/api/report/generate/status` behaves like a two-tier recovery API:
  - if `simulation_id` already maps to a completed report, it can recover that fact from disk
  - otherwise it falls back to `task_id`
  - if the backend has restarted, the old `task_id` likely returns `404`
- `/api/report/check/<simulation_id>` can still recover `has_report` / `report_id` / `report_status` from disk after restart, but it only unlocks interview when the report is fully completed.
- In practice, that recovery is ambiguous when multiple report folders share one `simulation_id`, because the current scan returns the first matching report rather than an explicitly preferred one.
- In other words, the API currently exposes at least two different definitions of "the report for this simulation":
  - first matching report folder
  - newest created report
- `/api/report/chat` is naturally restart-tolerant in a narrow sense:
  - it reconstructs state from `simulation_id`
  - reloads the saved report
  - depends on client-supplied `chat_history`
  - it does not depend on a surviving backend chat session
- In practice, chat inherits the same report-selection ambiguity:
  - after restart it may ignore a newer partial report
  - and fall back to an older empty or failed report
- `/api/simulation/<simulation_id>` and `/api/simulation/list` are restart-tolerant at the artifact layer because `SimulationManager` reloads persisted `state.json`.

## Lifecycle Walkthrough

1. `project_id` anchors extracted text, ontology, graph ID, and simulation requirement.
2. `simulation_id` anchors simulation artifacts and later the live runtime process.
3. `/api/report/generate` creates both `task_id` and `report_id`.
4. The daemon thread runs `ReportAgent.generate_report(...)` under `report_id`.
5. Each completed section is flushed to disk immediately, so report durability advances ahead of task/session durability.
6. Post-report chat does not resume section-generation state; it starts a new request-scoped loop from saved report artifacts.

## What Restart Most Likely Means In Practice

- Restart during simulation prepare:
  - likely recoverable by reloading `simulation_id` and inspecting `state.json`
  - unclear whether partially written prepare artifacts are always self-consistent
- Restart during wait-command mode:
  - saved simulation artifacts survive
  - the live OASIS environment likely needs to be relaunched or reattached
- Restart during report generation:
  - finished sections and logs survive
  - in-flight section-loop state is lost
  - polling by old `task_id` fails even if `report_id` artifacts already exist
  - lookup by `simulation_id` may still drift to an older report if several report folders exist
- Restart during report chat:
  - prior backend-held loop state is irrelevant because chat is request-scoped
  - follow-up chat can continue only via saved report artifacts plus client-supplied history

## Inferences

- `MiroFish` is artifact-first, not session-first.
- The most resumable layer is the filesystem artifact layer, not the coordinator layer.
- Restart safety is asymmetric:
  - durable identifiers are `project_id`, `simulation_id`, and `report_id`
  - non-durable coordination identifiers are `task_id` and any live in-memory loop state
- The API surface partially exposes that asymmetry, but not perfectly:
  - some routes recover from disk cleanly
  - some routes still assume the in-memory coordinator survived
- Session ownership is intentionally split:
  - simulations own runtime state
  - reports own analytical artifacts
  - tasks own UI polling state
  - chat leaves conversational continuity to the client
- This makes the system easy to inspect, but it also creates multiple truth surfaces that can drift.
- The strongest operational pattern is:
  - keep control flow ephemeral
  - externalize milestones quickly
  - reconstruct later from artifacts instead of reviving rich in-memory sessions
- The restart probe shows one more asymmetry:
  - `report_id` is a strong recovery handle
  - `simulation_id` is currently a weak recovery handle when multiple reports exist
- The system therefore does not yet have one canonical simulation-scoped report notion.

## Weak Spots

- `task_id` is useful for polling but is not a durable session handle.
- `report_id` is durable, but unfinished report loops do not persist a resumable checkpoint.
- Chat and report generation share artifacts, not one continuous session state.
- Backend cleanup is good for process hygiene, but probe-style backends can accidentally kill valuable simulation sessions.
- `get_report_by_simulation()` is simple, but it weakly defines “the active report session” when multiple reports exist.
- Restart after partial report generation can leave the user in an awkward middle state:
  - sections and logs exist
  - `task_id` is dead
  - no explicit resume endpoint exists
- The current API also lacks a first-class state for:
  - "partial report artifacts exist"
  - "report generation used to be active but is no longer running"
  - "safe to continue with post-report chat using the latest saved artifacts"
- The current report-selection logic can also create a misleading state:
  - a newer partial report exists
  - a newer completed report may also exist
  - but by-simulation lookup still returns an older failed report
- This is not just a backend storage quirk; it creates divergent user paths:
  - history-style navigation can point to the newest report id
  - chat/check/by-simulation can still bind to an older failed report

## Open Questions

- Does a restarted backend offer enough information for the UI to distinguish:
  - "unfinished but inspectable report artifacts exist"
  - "report generation is still active"
  - "report generation died and needs a new run"
- Should task state become durable, or should the UI rely only on `report_id` plus on-disk progress artifacts?
- Should report generation persist a richer loop checkpoint than completed sections plus coarse progress?
- Should chat sessions get a persistent backend ID, or is client-owned history the intended design?
- What should `get_report_by_simulation()` prefer when several reports exist:
  - latest created report
  - latest completed report
  - latest report with markdown content
  - a status-priority order such as `completed > generating/planning > failed`
- Should the codebase standardize on one policy everywhere, or intentionally expose two different concepts:
  - latest attempt
  - latest successful report

## Next Action

- Treat session management in `MiroFish` as a restart-asymmetric four-layer system with one more subtle issue:
  - artifact survival is real
  - artifact selection by `simulation_id` is currently unstable
- The next best validation is to compare recovery policies:
  - direct `report_id` retrieval
  - by-simulation retrieval
  - report check endpoint
  - post-restart chat reconstruction
