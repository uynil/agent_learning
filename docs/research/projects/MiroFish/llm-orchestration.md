---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: run one full report generation and inspect agent_log.jsonl for actual tool mix, interview timing, and fallback behavior
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

## Inferences

- The project intentionally mixes a central wrapper with specialized direct clients because different stages need different reliability strategies.
- LLM scheduling is phase-based, not continuous: ontology, profile generation, config generation, outline planning, section synthesis, and chat are separate call sites.
- The most constrained call sites are the report flows, where output format and tool usage are tightly controlled.
- The section writer is the clearest “agent core” in MiroFish; earlier phases are mostly structured generation services, not autonomous multi-step agents.
- The orchestration philosophy is: use strict JSON for setup, use bounded tool loops for research, and use lighter chat loops only after a report already exists.
- Report generation is hierarchical rather than flat:
  - plan the report from graph context
  - research one section at a time
  - compress completed sections into bounded downstream context
  - expose a lighter post-report QA surface
- The tool layer is epistemically split:
  - `insight_forge`, `panorama_search`, and `quick_search` operate on graph memory
  - `interview_agents` crosses into the live simulation runtime
- Live interview evidence is opportunistic rather than guaranteed, because the outer report orchestration does not gate on simulation readiness before section generation begins.

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
- Report persistence is incremental, so orchestration progress is externalized into report artifacts instead of kept only in memory.
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
- Whether a report-generation preflight should distinguish:
  - graph-ready
  - simulation-started
  - interview-ready
- Whether `agent_log.jsonl` shows true tool diversity in practice, or whether the loop mostly oscillates between one or two favorite tools.

## Next Action

- Inspect one real report run to see:
  - which tools are actually chosen per section
  - whether interviews happen only after readiness
  - where final-answer repair and forced closure are triggered
