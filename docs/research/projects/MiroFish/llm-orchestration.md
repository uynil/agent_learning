---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: compare direct `report_id` recovery with simulation-scoped recovery after restart, because orchestration artifacts survive but lookup policy is currently unstable
---

# LLM Orchestration

## Observed Facts

- The shared LLM wrapper is `backend/app/utils/llm_client.py`, which uses the OpenAI-compatible API shape and strips `<think>...</think>` blocks from responses.
- Model config comes from environment variables: `LLM_API_KEY`, `LLM_BASE_URL`, and `LLM_MODEL_NAME`.
- Several services bypass the shared wrapper and call `openai.OpenAI` directly, especially the configuration and persona generators.
- `OntologyGenerator` uses one strict JSON-producing call to create the graph schema, and truncates source text to 50,000 characters before sending it to the model.
- `SimulationConfigGenerator` splits generation into time config, event config, and batched agent configs. Its retry ladder lowers temperature on each retry and tries to repair truncated JSON before failing.
- `OasisProfileGenerator` enriches entity context, can query Zep for more background, and generates persona data with JSON-mode retries.
- `ReportAgent.plan_outline()` uses `chat_json` at low temperature and falls back to a default outline if parsing fails.
- `ReportAgent._generate_section_react()` is the most agentic loop in the codebase:
  - up to 5 iterations
  - one tool call per turn
  - at least 3 tool calls before final answer
  - hard conflict handling if the model outputs tool calls and final content together
  - forced final-answer fallback when the loop exhausts its budget
- That section loop keeps its own explicit runtime state:
  - `messages`
  - `tool_calls_count`
  - `used_tools`
  - `conflict_retries`
  - truncated `previous_sections`
  - tiny `report_context`
- `ReportAgent.generate_report()` is a hierarchical scheduler:
  - create report folder and logs
  - plan outline once
  - generate sections sequentially
  - save each section immediately
  - assemble the final report at the end
- Section generation reuses prior section content, but truncates each completed section to 4000 characters before passing it into the next section prompt.
- Each section also passes a small `report_context` string into `insight_forge`, so sub-question generation sees the section title and simulation requirement.
- `ReportAgent.chat()` is a lighter follow-up mode with at most 2 tool calls and a truncated report context.
- `ReportAgent.chat()` rebuilds context from the saved report, truncates report content to 15000 characters, and only keeps the last 10 history turns.
- `insight_forge` is itself a multi-step LLM-assisted retrieval path:
  - generate sub-queries with `chat_json`
  - search graph edges for each sub-query
  - expand related entities and relationship chains
- `interview_agents` is also multi-stage:
  - load profiles from persisted simulation artifacts
  - use LLM to select interview targets
  - use LLM to generate interview questions
  - call `SimulationRunner.interview_agents_batch(...)` against the live environment
  - use LLM again to summarize interview responses
- The interview path is intentionally prompt-shaped to suppress tool use on the simulated agents and force direct text answers.
- `/api/report/generate` launches report generation in a background thread after checking graph/project metadata, but it does not verify that the simulation environment is already interview-ready.
- One validated live report run (`report_7e34e64b29aa`) finished end-to-end in about 316 seconds and produced 3 sections plus `full_report.md`.
- In that run, the actual section tool sequences were:
  - section 1: `panorama_search -> insight_forge -> interview_agents`
  - section 2: `panorama_search -> insight_forge -> interview_agents -> quick_search`
  - section 3: `panorama_search -> insight_forge -> interview_agents`
- The measured tool latencies in that run were roughly:
  - `panorama_search`: 1-2 seconds
  - `insight_forge`: about 7 seconds
  - `interview_agents`: about 48-57 seconds
  - `quick_search`: about 1 second
- All 3 section-level interview tool calls succeeded structurally:
  - batch IPC returned `completed`
  - 4 agents were selected
  - interview summaries were returned to the report loop
- All 3 of those interview summaries were still evidence-empty at the report layer:
  - each per-agent block showed no Twitter reply
  - each per-agent block showed no Reddit reply
- Follow-up inspection shows that the underlying batch IPC response files were not empty. In the validated single-platform run, the runner returned substantive answers keyed as bare agent IDs such as `"0"` and `"3"`.
- The empty report-layer interview summaries therefore come from an adapter mismatch, not necessarily from a simulation-side failure:
  - `zep_tools.py` parses results as `twitter_{agent_id}` / `reddit_{agent_id}`
  - `run_twitter_simulation.py` returns batch results as `{agent_id: {...}}`
  - the report tool drops the real answers and renders placeholders instead
- Section 2 shows an additional recovery path: after an empty interview result, the model called `quick_search` once more before closing the section.
- Section 2 also exposed a prompt/code contract mismatch:
  - the final LLM response had no tool call
  - it also did not include the expected `Final Answer:` prefix
  - the backend accepted that response as final section content anyway
- During the long interview waits, `progress.json` stayed stuck on older state while `agent_log.jsonl` and backend logs kept advancing.

## Inferences

- The project intentionally mixes a central wrapper with specialized direct clients because different stages need different reliability strategies.
- LLM scheduling is phase-based, not continuous: ontology, profile generation, config generation, outline planning, section synthesis, and chat are separate call sites.
- The most constrained call sites are the report flows, where output format and tool usage are tightly controlled.
- The section writer is the clearest “agent core” in MiroFish; earlier phases are mostly structured generation services, not autonomous multi-step agents.
- The orchestration philosophy is: use strict JSON for setup, use bounded tool loops for research, and use lighter chat loops only after a report already exists.
- The agent loop is therefore session-local, not global:
  - planning has its own prompt state
  - each section gets a fresh bounded loop
  - chat starts a separate lighter loop later
- Report generation is hierarchical rather than flat:
  - plan the report from graph context
  - research one section at a time
  - compress completed sections into bounded downstream context
  - expose a lighter post-report QA surface
- The tool layer is epistemically split:
  - `insight_forge`, `panorama_search`, and `quick_search` operate on graph memory
  - `interview_agents` crosses into the live simulation runtime
- Live interview evidence is opportunistic rather than guaranteed, because the outer report orchestration does not gate on simulation readiness before section generation begins.
- The report loop's real-world behavior is more prescribed than the static code alone suggests. In the validated run, the first three tools were nearly canonical rather than freely explored.
- `interview_agents` currently behaves like a high-latency, high-coupling probe whose structural success can be masked as evidence failure by a response-schema mismatch.
- The section loop degrades gracefully when live interview evidence is empty: it still produces coherent sections using graph retrieval and prior reasoning.
- The validated run no longer supports the stronger claim that live interviews were intrinsically low-yield. The more accurate claim is that report-time orchestration failed to decode the single-platform interview payload.
- The extra `quick_search` in section 2 suggests the model sometimes notices that the interview branch did not add enough confidence, but the overall closure still remains graph-led.
- The orchestration layer tolerates final-answer format drift more than the prompt contract implies, because a no-tool output can still be accepted without the `Final Answer:` marker.
- Orchestration is more durable at the artifact layer than at the loop layer: section outputs and logs persist, but loop-local state does not.
- The API layer exposes that durability split unevenly:
  - report artifacts can be rediscovered by `simulation_id` / `report_id`
  - task polling depends on the in-memory `TaskManager`
  - post-report chat reconstructs a new short loop from saved artifacts rather than resuming the old section loop

## Call Sites

- `OntologyGenerator`: strict JSON schema for entity and edge types.
- `SimulationConfigGenerator`: time config, event config, and batch agent configs.
- `OasisProfileGenerator`: persona enrichment from entity context.
- `ReportAgent.plan_outline()`: JSON outline with 2-5 sections.
- `ReportAgent._generate_section_react()`: iterative tool-using section writer with section-local memory and tool-budget enforcement.
- `ReportAgent.chat()`: short-answer chat with a small tool budget.

## Tool Roles

- `insight_forge`
  - deep query decomposition
  - relationship and entity expansion
  - best suited for section-level analytical synthesis
- `panorama_search`
  - whole-graph breadth
  - active vs historical fact separation
  - best suited for timeline and world-state framing
- `quick_search`
  - narrow fact verification
  - best suited for spot checks and cheap follow-up retrieval
- `interview_agents`
  - live first-person evidence from simulated agents
  - highest-latency and highest-runtime-coupling tool in the stack

## Scheduling Notes

- Planning uses low temperature and a single JSON parse path.
- Section generation uses a ReAct loop with up to 5 iterations, at least 3 tool calls, and forced final-answer fallback.
- Chat uses a lighter loop with up to 2 iterations and at most 2 tool calls.
- Simulation config generation uses a retry ladder that lowers temperature on each attempt and attempts JSON repair if the model truncates output.
- Chat and section writing use the same underlying tools, but the allowed search depth and context budget are intentionally different.
- Section scheduling is sequential, not parallel: later sections inherit bounded context from earlier sections, but each section has its own fresh tool loop.
- The section loop behaves like a finite-state machine:
  - request next move
  - execute one tool or accept closure
  - append observation or repair prompt
  - continue until valid closure or forced fallback
- Report persistence is incremental, so orchestration progress is externalized into report artifacts instead of kept only in memory.
- In practice, that incremental persistence has different trust levels:
  - `agent_log.jsonl` is the fine-grained execution trace
  - `section_XX.md` is the durable user-facing output
  - `progress.json` is a lagging coarse progress indicator during long-running tool calls
- Task polling is more fragile than report inspection:
  - `/api/report/generate/status` can recover a completed report from disk if `simulation_id` resolves
  - otherwise it falls back to `task_id`
  - after backend restart, unfinished work may still leave report artifacts on disk while the original `task_id` no longer resolves
- A real restart probe now confirms that split:
  - old `task_id` immediately failed after restart
  - direct `GET /api/report/<report_id>` still reconstructed the interrupted planning-stage report
  - simulation-scoped lookup routes returned a different older failed report instead
- The reason is now concrete rather than abstract:
  - report-manager lookup is first-match
  - simulation-history lookup is latest-created
  - chat inherits the manager-side first-match policy because it only receives `simulation_id`
- LLM usage is concentrated around translation tasks:
  - text -> ontology
  - entities -> personas
  - scenario -> simulation parameters
  - evidence -> report sections
  - report + retrieval -> concise chat answer

## Inferences About Model Risk

- Because some stages require exact JSON, the code is optimized for parseability over stylistic freedom.
- The report layer is the highest-risk part for hallucination, so it compensates with tool requirements, raw evidence reuse, and postprocessing.
- The biggest integration risk is not one bad model answer in isolation, but the handoff across multiple phases with different prompt contracts and different reliability code.
- The report layer's highest integration risk is not only hallucination but runtime mismatch: if `interview_agents` is chosen before the simulation is ready, the section loop can only recover through best-effort retries or degraded evidence.

## Open Questions

- Which model is expected in production versus local development.
- Whether the direct `OpenAI` usage should eventually be consolidated behind `LLMClient`.
- Whether the 3-tool minimum materially improves report quality, or just increases cost and latency.
- Whether the forced-final-answer fallback produces acceptable sections when tools underperform.
- How often real report runs actually choose `interview_agents` versus staying graph-only.
- Whether `zep_tools.py` should accept both single-platform (`"0"`) and parallel (`"twitter_0"`) batch result schemas.
- Whether the single-platform runners should instead normalize their own output to the dual-platform schema documented by the public API.
- Whether the current loop state should ever become resumable, or whether the design intentionally treats loops as disposable once artifacts are flushed.
- Whether a report-generation preflight should distinguish:
  - graph-ready
  - simulation-started
  - interview-ready
- Whether `/api/report/generate/status` should explicitly model the state:
  - partial report artifacts exist
  - no active in-memory task survives
  - safe to inspect but not truly resumable
- Whether report orchestration should treat `report_id` as the primary post-start handle, with `simulation_id` only as a discovery key rather than as the canonical recovery key.
- Whether chat/report-check/by-simulation routes should accept `report_id` explicitly once a generation attempt has started.
- Whether `agent_log.jsonl` shows true tool diversity in practice, or whether the loop mostly oscillates between one or two favorite tools.
- Whether the missing `Final Answer:` prefix in section 2 was a one-off drift or a recurring pattern under live latency.

## Next Action

- Inspect one real report run to see:
  - where batch interview payloads change shape across single-platform vs parallel runners
  - whether the next report run still repeats the same `panorama -> insight -> interview` rhythm once the payload is decoded correctly
  - whether final-answer normalization should be made more explicit in logs and docs
