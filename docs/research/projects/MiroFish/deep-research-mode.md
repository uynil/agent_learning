---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: model the section loop as a real state machine and verify how much deep-research continuity is preserved between planning, section writing, and post-report chat
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
- The section loop also has explicit repair transitions:
  - if the model returns `None`, ask it to continue
  - if it mixes tool call and final answer, reject and retry
  - if it closes too early, reject and require more tools
  - if it exceeds budget, force finalization
  - if it returns plain text without `Final Answer:` after enough tools, accept it anyway
- If the model outputs both a tool call and a final answer in one response, the code rejects or truncates the response and forces the loop back into a valid shape.
- Each section inherits bounded memory from previous sections, but only through truncated prose rather than a structured claim ledger.
- The single-platform live interview path has now been validated directly outside the report agent:
  - wait-command mode can stay alive under a persistent backend
  - interview answers come back through IPC
  - interview traces are written to the platform database
- That validated path still has a warm-up requirement before the environment becomes truly interview-ready, so deep-research behavior depends on runtime readiness as well as prompt/orchestration logic.
- `/api/report/generate` does not wait for interview readiness before starting report generation, so interview-backed evidence is currently an opportunistic enhancement rather than a guaranteed precondition.
- A full live report run now confirms that this deep-research mode is operational, not just designed in code:
  - 3 sections were generated successfully
  - the total report took about 316 seconds
  - all sections completed without manual recovery
- In that run, the section-level research loops were highly regular:
  - every section used `panorama_search`
  - every section used `insight_forge`
  - every section used `interview_agents`
  - one section also used `quick_search`
- The live interview branch was the dominant latency source in every section, adding roughly 48-57 seconds each time.
- The interview branch still looked empty in all 3 sections, but raw IPC responses show that single-platform agents did in fact generate substantive answers.
- The current weakness is therefore not purely “evidence level”. It is also an integration-level loss between the live interview transport and the report tool that consumes it.
- Even with those empty interviews, the report still produced coherent section prose and a completed final report.

## Inferences

- This is a bounded deep-research mode embedded inside report generation.
- It is strongest when the question needs synthesis across graph evidence and live simulated viewpoints.
- The system behaves more like a research report engine than a general autonomous analyst because each section has explicit limits, and the agent is forced to close.
- The agent loop is not one monolithic planner. It is a per-section finite-state machine with reset boundaries between sections.
- The deep-research behavior is real, but it lives at the section level, not at the whole-report or whole-project level.
- The live interview capability is not just designed in theory; it is operational, but its usefulness is coupled to the runtime contract of the simulation layer.
- The research loop is hybrid rather than uniform:
  - graph tools are always available once the graph exists
  - live interview evidence is only available when the simulation has crossed its readiness boundary
- ReportAgent is doing genuine evidence acquisition, but it still lacks a whole-report planner/reviewer split or an explicit hypothesis tracker.
- The live run still suggests a graph-led deep-research system, but that conclusion is now provisional because some of the interview evidence was dropped before the section writer saw it.
- The bounded deep-research loop is resilient to weak live evidence, but that resilience also means low-yield interviews can silently become expensive no-op steps.
- In the current single-platform setup, those “low-yield interviews” are at least partly artificial, because valid answers can be transformed into placeholders by the adapter layer.

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
- The validated run shows those epistemic roles are real in orchestration, but not equally visible in report-time evidence:
  - breadth and decomposition reliably return useful graph evidence
  - live viewpoints can generate substantive answers, but the current single-platform adapter can hide them from the section writer

## Loop Mechanics

1. Build a fresh section prompt from:
   - report title/summary
   - simulation requirement
   - current section title
   - truncated previous sections
2. Initialize loop state:
   - `messages`
   - `tool_calls_count`
   - `used_tools`
   - `conflict_retries`
3. Ask the LLM for exactly one next move.
4. If the move is a tool call, execute one tool and append an observation message.
5. If the move is an invalid closure, append a repair message and stay in loop.
6. If the move is a valid closure, save the section and reset the loop for the next section.

Deep-research continuity is therefore artifact-mediated, not loop-mediated:
- sections inherit prior prose
- they do not inherit prior internal loop state
- chat starts a lighter new loop rather than resuming section generation

## Why It Is Still Bounded

- The loop is section-scoped, not open-ended across arbitrary research tasks.
- There is no explicit planner/reviewer split across the whole report.
- There is no persistent hypothesis tracker or claim ledger visible in the current code.
- The system does not appear to branch into multiple independent research tracks.
- Previous section context is truncated before reuse, so long-horizon research memory is intentionally compressed.
- The chat surface is a lighter follow-up loop, not an unbounded continuation of the report-generation research state.
- The orchestration does not guarantee that the highest-value tool (`interview_agents`) is actually ready at the moment a section wants it.
- The current boundedness does not include a strong “stop using low-yield tools” heuristic, so a section can keep paying the latency cost of interviews even when prior interview evidence has been empty.
- The current boundedness also means there is no server-owned long-lived “research session” spanning planning, section writing, and chat as one continuous agent mind.

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
- Whether interview evidence materially changes section conclusions compared with graph-only retrieval once the single-platform parsing mismatch is removed.
- In the current live run, interview evidence did not visibly change the final prose. Is that a real product property, or mostly an artifact of the adapter bug?
- Whether the report agent currently has enough readiness signaling to know when the live simulation is actually interview-ready versus merely launched.
- Whether full report runs actually use all four tools in a meaningfully differentiated way, or whether tool diversity is more prescribed than real.
- Whether the system should persist an explicit section-loop checkpoint if true resumable deep research becomes important.

## Next Action

- Build on the validated interview path by comparing:
  - graph-only sections
  - graph + interview sections
  - sections whose interviews return empty evidence vs sections with real replies
  - sections that barely satisfy the 3-tool minimum vs sections that use richer tool diversity
  - section-loop state vs post-report chat state
