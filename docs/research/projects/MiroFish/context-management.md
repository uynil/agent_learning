---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: decide whether recovery should pivot from `simulation_id` to `report_id` once multiple report artifacts coexist, because current simulation-scoped lookup can miss newer partial artifacts
---

# Context Management

## Observed Facts

- Uploaded source documents are stored under the project folder, and extracted text is persisted separately.
- The project model stores ontology, graph ID, simulation requirement, and file metadata in `project.json`.
- Simulation preparation persists `state.json`, `reddit_profiles.json` or `twitter_profiles.csv`, and `simulation_config.json`.
- Live simulation state persists in `run_state.json`.
- The live simulation also persists per-platform `actions.jsonl` streams and filesystem-based IPC folders for interview commands and responses.
- Report generation persists `meta.json`, `outline.json`, `progress.json`, `section_XX.md`, `full_report.md`, `agent_log.jsonl`, and `console_log.txt`.
- Zep paging is explicit and bounded: nodes and edges are fetched page by page with retry logic.
- The report agent truncates prior section context to 4000 characters per section before feeding it into later section prompts.
- Chat mode truncates the generated report context to 15000 characters and only keeps the last 10 messages of history.
- Planning also uses a compressed context view:
  - graph statistics
  - entity-type list
  - total entities
  - first 10 related facts
- Report generation saves each completed section immediately, so the report folder acts as a rolling research memory rather than a single end-of-run dump.
- `interview_agents` preserves full per-agent responses plus extracted quotes and a synthesized summary, so one tool call carries both rawer evidence and compressed evidence.
- `SimulationConfigGenerator` caps assembled source context at 50,000 characters and uses smaller caps for sub-prompts like time config and event config.
- `ZepGraphMemoryUpdater` batches meaningful activities by platform before writing them back into graph memory.
- `TaskManager` is not part of the durable context model; its progress state is in-memory only.
- In validated single-platform live runs, the strongest evidence of actual simulation work came from `twitter_simulation.db` and `simulation.log`, not from `run_state.json`.
- In validated single-platform live runs, `env_status.json` becomes useful only after a warm-up window; before that, `/start` has already returned but IPC is not yet ready.
- Successful interviews are persisted in the platform database and can be read back through `/api/simulation/interview/history`.
- Single-platform `env_status.json` currently stores only `status` and `timestamp`, while the backend status reader expects richer per-platform fields.
- `/api/report/generate` can start a report before the simulation is interview-ready, so the availability of live interview context is timing-dependent rather than guaranteed by the outer workflow.
- One full live report run now confirms the complete report artifact set:
  - `meta.json`
  - `outline.json`
  - `progress.json`
  - `section_01.md`
  - `section_02.md`
  - `section_03.md`
  - `full_report.md`
  - `agent_log.jsonl`
  - `console_log.txt`
- In that run, `progress.json` lagged badly during long `interview_agents` calls. It stayed at section-2-complete / 66% even while section 3 was already issuing tools and waiting on IPC.
- `agent_log.jsonl` remained current throughout the same period and captured:
  - per-section start/completion
  - each tool call and tool result
  - each LLM response
  - total report completion time
- The final report sections did not embed raw interview transcripts or agent identifiers, even though those detailed interview summaries existed in `agent_log.jsonl`.
- The report context that survives into later artifacts is therefore more compressed than the execution trace: the logs preserve process richness, but the final markdown preserves only the synthesized conclusions.
- Follow-up inspection of the raw IPC response files adds another boundary fact:
  - single-platform batch interview artifacts preserve substantive answers under bare agent-id keys
  - the report adapter rewrites that into empty dual-platform placeholders because it expects a different schema
  - the loss therefore happens between IPC response storage and report-tool decoding, not inside the simulation environment itself

## Inferences

- The project treats context as a layered cache:
  - raw source text
  - graph memory
  - simulation artifacts
  - report artifacts
  - live chat context
- There are actually two different durability classes:
  - durable: files on disk and graph memory in Zep
  - ephemeral: async task progress, live process state not yet written, and current request context
- There are also several different context scopes:
  - planning scope
  - section scope
  - report artifact scope
  - chat scope
  - simulation runtime scope
- Restart behavior follows those context scopes unevenly:
  - report artifact scope survives well
  - simulation artifact scope survives well
  - task scope does not survive
  - chat scope only survives if the client can replay it
- The restart probe shows one more unevenness inside report context itself:
  - direct `report_id` access preserved the interrupted report's outline and progress snapshot
  - simulation-scoped lookup did not reliably surface that same partial context
- Most of the system’s reliability comes from serialization boundaries, not from one huge in-memory state object.
- The truncation strategy is deliberate and consistent with long-running prediction/report jobs.
- The system has multiple runtime state surfaces, but they are not equally authoritative.
- In single-platform mode, the backend's nominal progress surface (`run_state.json`) is weaker than the DB/log surfaces that record what actually happened.
- Report artifacts are doing double duty:
  - persistence for user-visible outputs
  - compressed memory for follow-up chat and later sections
- The live interview path injects runtime-only context into the report loop, but that context is opportunistic because readiness is not guaranteed before report generation starts.
- The report artifact layer has now been empirically stratified by authority:
  - `agent_log.jsonl` is the best source for execution truth
  - `section_XX.md` and `full_report.md` are the best sources for durable synthesized output
  - `progress.json` is a delayed user-facing status indicator rather than a precise execution ledger
- Mis-decoded interview summaries still consume a large amount of transient context inside the section loop, but the actual single-platform answers never make it into that context window.

## Context Boundaries

## Ingestion boundary

- File parsing normalizes PDFs, Markdown, and text before graph construction.
- `TextProcessor` normalizes line breaks and compresses extra whitespace.

## Graph boundary

- `ZepEntityReader` and `zep_paging` keep graph reads bounded and retry-safe.
- `InsightForge` further reduces context by generating subqueries first and deduplicating results.
- Ontology generation truncates the source text before prompt submission, but graph construction still operates on the full extracted text stream.
- Planning later re-compresses graph state again by sampling only a small fact window instead of replaying the whole graph.

## Simulation boundary

- `SimulationConfigGenerator` caps its assembled context at 50000 characters.
- It separately truncates time and event prompts for focused generation.
- Simulation state is split across config files, run-state files, per-platform action logs, and optional graph-memory updates.
- IPC interview requests and responses are persisted as files, which makes the live interaction boundary explicit and inspectable.
- In single-platform mode, the simulation boundary is effectively spread across four artifact layers:
  - `env_status.json` for coarse liveness
  - `run_state.json` for backend-facing progress
  - `simulation.log` for execution trace
  - platform SQLite DB for durable action/interview evidence
- Those four layers are currently only partially aligned.

## Report boundary

- Section generation only gets prior sections, not the entire report.
- Prior section reuse is lossy by design: each completed section is capped at 4000 characters before being forwarded.
- Section-local reasoning state is also intentionally non-persistent:
  - later sections inherit prior prose
  - they do not inherit prior `messages`
  - they do not inherit prior `used_tools`
  - they do not inherit an explicit claim ledger
- The report assembly step rebuilds the full markdown from saved sections and then cleans heading levels.
- Each completed section is saved immediately, so the report artifact evolves incrementally rather than only at the end.
- Structured agent logs and console logs preserve how the report was produced, not just the final markdown.
- `report_context` handed to `insight_forge` is tiny compared with the accumulated report; it only carries the section title and simulation requirement.
- Chat mode does not resume the full section-generation state. It reconstructs a smaller context window from saved report markdown plus the latest chat turns.
- The report artifact layer is therefore a replay source, not a continuation of the section loop itself.
- The validated live run shows an additional memory distinction inside the report boundary:
  - `agent_log.jsonl` keeps the large interview summaries and per-turn reasoning trace
  - `section_XX.md` keeps only the final narrative synthesis
  - later sections inherit prior prose, not the richer logged evidence trail
- Restart would therefore restore the report mostly as:
  - saved outputs
  - saved traces
  - not the exact active reasoning state inside an unfinished section loop

## Interaction boundary

- Chat answers are shaped by the saved report and recent chat turns, not by the full simulation history.
- Chat history is client-supplied rather than server-persisted, so conversational continuity is treated as request input, not backend state.
- Post-restart chat continuity is therefore conditional:
  - saved report artifacts provide the server-owned base context
  - only the client can restore the missing conversational thread
- In practice, that continuity is also selection-sensitive:
  - if chat reloads the wrong report for a shared `simulation_id`
  - then even valid client-supplied history cannot recover the intended analytical state
- The restart probe shows that this is not hypothetical:
  - direct `report_id` access preserved the partial planning artifact
  - simulation-scoped chat ignored it and reconstructed from an older empty/failed report path
- Interview mode crosses back into the live simulation boundary through IPC instead of only querying persisted artifacts.
- The interview path is real and validated, but it depends on timing-sensitive runtime readiness that is not fully expressed by the current API response from `/start`.
- The internal report-generation interview path and the post-report user interview surface are not the same context boundary:
  - internal sections can call `interview_agents` during report generation
  - the public `/check/<simulation_id>` route only unlocks the UI interview affordance after report completion

## Weak Spots

- `TaskManager` state disappears on backend restart, even though the underlying artifacts may still exist.
- The system has many durable files, but not all of them are summarized into one canonical resumable manifest.
- Chat reuses the saved report as a compressed proxy for the simulation, which may hide lower-level evidence unless tools are called again.
- `run_state.json` is only as good as the monitor inputs feeding it; when `actions.jsonl` is missing, it becomes a weak proxy for true runtime progress.
- `env_status.json` is overloaded as both liveness signal and readiness hint, but in single-platform mode it does not expose enough detail for the backend to distinguish "process launched" from "fully interview-ready".
- The report loop does not appear to persist a claim ledger or cited-evidence index, so later sections only inherit prose, not an explicit map of what has already been proven.
- When live interviews return empty replies, the system still persists a large “successful” interview artifact in the log layer, but that artifact contributes little durable evidence to the final report layer.
- Planning and chat are both intentionally context-thin compared with the section loop, which keeps prompt sizes bounded but also makes cross-session continuity shallow.

## Open Questions

- Whether the 15000-character chat cap is enough for large reports with many sections.
- Whether section-level truncation should also preserve a small evidence map or summary of what has already been cited.
- Whether the live simulation logs need a formal schema document for easier downstream consumers.
- Whether a backend restart during graph build or report generation leaves enough metadata to resume cleanly instead of just re-reading artifacts manually.
- Whether the backend should privilege `simulation.log` / SQLite over `actions.jsonl` when single-platform mode is active.
- Whether runtime readiness should be represented as its own explicit artifact instead of being inferred from `env_status.json`.
- Whether report generation should record when a live-interview tool was skipped or degraded because the environment was not ready.
- Whether `progress.json` should be updated at tool-call granularity, or explicitly documented as a coarse UI progress surface.
- Whether the report folder should persist a lighter section-level evidence ledger so later analysis does not have to reconstruct everything from `agent_log.jsonl`.
- Whether report-time tools should persist both raw IPC payloads and normalized payloads so schema drift is easier to diagnose.
- Whether the system should add a resumable “report loop checkpoint” file instead of only completed sections plus coarse progress.

## Next Action

- Use one real report run to compare the trustworthiness of:
  - `progress.json`
  - `agent_log.jsonl`
  - `console_log.txt`
  - `section_XX.md`
  - `full_report.md`
  - `simulation.log`
  - platform SQLite interview records
  - raw IPC response JSON
  - client-supplied chat history vs server-owned artifacts
