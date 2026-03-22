---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: run a full report against a warm simulation and compare graph-only sections with sections that actually use live interviews
---

# Deep Research Mode

## Observed Facts

- The report agent does real multi-step evidence gathering, not just one-shot summarization.
- `insight_forge` decomposes a query into subquestions before searching.
- `panorama_search` retrieves all nodes and edges and separates active facts from historical or expired facts.
- `quick_search` provides a narrow verification path.
- `interview_agents` sends IPC commands to the live OASIS environment and reads back actual agent responses.
- Report generation is hierarchical:
  - first plan a 2-5 section outline from graph context
  - then run bounded section-level research loops
  - then expose a lighter post-report chat surface
- Section generation requires a minimum amount of tool use and can continue iterating until the model emits a final answer.
- The UI exposes these tool calls and responses as structured report logs.
- The section loop is explicitly bounded in code:
  - at most 5 iterations
  - at least 3 tool calls
  - at most 5 tool calls
  - one tool call per turn
  - forced final-answer fallback if the loop does not close cleanly
- If the model outputs both a tool call and a final answer in one response, the code rejects or truncates the response and forces the loop back into a valid shape.
- Each section inherits bounded memory from previous sections, but only through truncated prose rather than a structured claim ledger.
- The single-platform live interview path has now been validated directly outside the report agent:
  - wait-command mode can stay alive under a persistent backend
  - interview answers come back through IPC
  - interview traces are written to the platform database
- That validated path still has a warm-up requirement before the environment becomes truly interview-ready, so deep-research behavior depends on runtime readiness as well as prompt/orchestration logic.
- `/api/report/generate` does not wait for interview readiness before starting report generation, so interview-backed evidence is currently an opportunistic enhancement rather than a guaranteed precondition.

## Inferences

- This is a bounded deep-research mode embedded inside report generation.
- It is strongest when the question needs synthesis across graph evidence and live simulated viewpoints.
- The system behaves more like a research report engine than a general autonomous analyst because each section has explicit limits, and the agent is forced to close.
- The deep-research behavior is real, but it lives at the section level, not at the whole-report or whole-project level.
- The live interview capability is not just designed in theory; it is operational, but its usefulness is coupled to the runtime contract of the simulation layer.
- The research loop is hybrid rather than uniform:
  - graph tools are always available once the graph exists
  - live interview evidence is only available when the simulation has crossed its readiness boundary
- ReportAgent is doing genuine evidence acquisition, but it still lacks a whole-report planner/reviewer split or an explicit hypothesis tracker.

## Why It Counts as Deep Research

- It first plans the report structure from graph context.
- It decomposes the question.
- It mixes broad and narrow retrieval.
- It incorporates live interview evidence from the simulation environment.
- It preserves evidence in logs and report artifacts.
- It iterates until enough material has been gathered for a section-level synthesis.
- It uses different retrieval modes for different epistemic jobs:
  - `insight_forge` for structured decomposition
  - `panorama_search` for world-state breadth
  - `quick_search` for fact checking
  - `interview_agents` for first-person simulated viewpoints

## Why It Is Still Bounded

- The loop is section-scoped, not open-ended across arbitrary research tasks.
- There is no explicit planner/reviewer split across the whole report.
- There is no persistent hypothesis tracker or claim ledger visible in the current code.
- The system does not appear to branch into multiple independent research tracks.
- Previous section context is truncated before reuse, so long-horizon research memory is intentionally compressed.
- The chat surface is a lighter follow-up loop, not an unbounded continuation of the report-generation research state.
- The orchestration does not guarantee that the highest-value tool (`interview_agents`) is actually ready at the moment a section wants it.

## Practical Shape

1. Plan a short report outline from graph statistics and fact samples.
2. For each section, reuse bounded prior-section context.
3. Gather graph evidence with a mix of deep, broad, and quick retrieval.
4. Ask the live agents when first-person perspectives matter and the simulation is ready.
5. Synthesize the section and save it immediately.
6. Assemble the final report and expose a lighter post-report chat loop.

## Open Questions

- Whether the current evidence mix is enough for high-stakes forecast claims.
- Whether the product should add a claim-tracking layer or source matrix for stronger auditability.
- Whether the deep-research behavior should be exposed more explicitly in the UI rather than appearing as report generation.
- Whether the 3-tool minimum produces measurably better sections, or mainly serves as a safety rail.
- Whether interview evidence materially changes section conclusions compared with graph-only retrieval.
- Whether the report agent currently has enough readiness signaling to know when the live simulation is actually interview-ready versus merely launched.
- Whether full report runs actually use all four tools in a meaningfully differentiated way, or whether tool diversity is more prescribed than real.

## Next Action

- Build on the validated interview path by comparing:
  - graph-only sections
  - graph + interview sections
  - sections generated before the environment is ready vs after it is ready
  - sections that barely satisfy the 3-tool minimum vs sections that use richer tool diversity
