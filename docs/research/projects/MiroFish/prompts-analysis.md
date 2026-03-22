---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: compare live agent_log outputs against the report prompt contract, especially around tool diversity, interview timing, and final-answer repair
---

# Prompts Analysis

## Observed Facts

- The ontology prompt requires valid JSON and exactly 10 entity types, with `Person` and `Organization` fixed as the last two fallback types.
- The ontology prompt is explicitly about social-media-style simulation, not abstract theory.
- The simulation-config prompts are split into time, event, and agent prompts, each with hard schema expectations.
- The report prompts are organized as:
  - `PLAN_SYSTEM_PROMPT`
  - `PLAN_USER_PROMPT_TEMPLATE`
  - `SECTION_SYSTEM_PROMPT_TEMPLATE`
  - `SECTION_USER_PROMPT_TEMPLATE`
  - `CHAT_SYSTEM_PROMPT_TEMPLATE`
- The section prompt explicitly bans Markdown headings inside generated section bodies and forces `Final Answer:`-style completion.
- The section prompt also forces an evidence-first workflow:
  - must call tools
  - must cite agent behavior and speech
  - must translate mixed-language evidence into Chinese report prose
  - must not rely on unstated model knowledge
- The ReAct loop is partly implemented in prompts and partly implemented in code. Prompt text tells the model how to behave; Python enforces minimum tool usage, one-tool-per-turn, and final-answer repair.
- The interview API prepends a prefix that tells agents not to call tools and to answer directly in text.
- Prompted control is not limited to `report_agent.py`. `zep_tools.py` also contains dedicated prompt families for:
  - sub-query generation inside `insight_forge`
  - agent selection for interviews
  - interview-question generation
  - interview-summary synthesis
- The section prompt has a highly layered structure:
  - worldview and task framing
  - hard rules
  - formatting constraints
  - tool catalog and tool-usage advice
  - turn-taking protocol
  - content requirements
- The section loop feeds the model markdown-heavy tool observations, including structured interview reports, and then asks the model to turn them back into heading-free section prose.
- The code keeps nudging prompt behavior toward tool diversity by showing unused-tool hints after each observation.
- The report chat prompt is structurally different from the section prompt: it is report-first, concise, and tool-optional rather than tool-mandatory.

## Inferences

- Prompt design is highly procedural. The prompts do not just describe the task; they also define the execution contract.
- The report prompts are the clearest example of an evidence-first style. They require tool use, raw quotes, and Chinese-language translation of evidence.
- Prompt boundaries are used to compensate for model variance, especially in the section writer where format errors are actively corrected.
- MiroFish uses prompts as workflow governors, not just semantic instructions. The prompt is part of the runtime control surface.
- Prompt logic is split across multiple layers:
  - report planning and section writing prompts define the outer research behavior
  - tool-specific meta-prompts shape what counts as evidence inside those loops
  - API-layer prefixes constrain live simulated agents during interviews
- The prompt system is trying to solve two problems at once:
  - generate good report prose
  - keep the model operating like a bounded tool-using analyst instead of a freeform writer

## Prompt Families

- **Ontology prompts**
  - Force a graph schema that is usable by Zep.
  - Push the model toward real-world agent types instead of abstract concepts.

- **Simulation config prompts**
  - Shape time-of-day activity, event setup, and per-agent behavior.
  - Use Chinese daily-rhythm defaults unless the scenario suggests otherwise.

- **Report planning prompts**
  - Ask for a short outline in JSON.
  - Limit the report to 2-5 sections.

- **Section writing prompts**
  - Require at least 3 tool calls.
  - Encourage mixed tool usage: deep search, panorama search, quick search, and interviews.
  - Forbid internal section headings and rely on system-level assembly to add top-level markdown.
  - Frame the report explicitly as future prediction rather than present-tense analysis.
  - Require direct grounding in simulated events and agent utterances.

- **Tool-meta prompts**
  - `InsightForge` asks the model to decompose one analytical question into up to 5 smaller observable sub-questions.
  - Interview selection asks the model to choose a diverse and relevant subset of agents.
  - Interview-question generation asks for 3-5 short open questions.
  - Interview-summary synthesis asks for consensus, disagreement, and quotable lines.

- **Chat and interview prompts**
  - Keep responses concise.
  - Limit tool use.
  - In interview mode, suppress tool use completely.

## Prompt Pattern Notes

- The same notion appears repeatedly as future prediction rather than present-tense analysis.
- The report layer actively tells the model what not to do, which is a sign that format adherence matters more than open-ended creativity.
- The interview prefix is a practical guardrail: it reduces the chance that the agent will try to invoke tools during a manual response channel.
- The strongest prompt pattern in this repo is split declarative goal plus procedural constraints plus code enforcement.
- The prompt system is modular by function rather than by one giant universal prompt.
- `ReportAgent` uses prompt structure and code together to create a bounded research dialect:
  - prompt says “use tools, no headings, evidence first”
  - code says “one tool per turn, at least 3 tools, stop after 5”

## Function-to-Mechanism Mapping

- **Ontology prompt**
  - Function: produce a graph-compatible simulation ontology
  - Mechanism: exact-count constraints, fallback type requirements, and strict JSON output

- **Simulation config prompts**
  - Function: turn scenario and entities into runnable time/event/activity parameters
  - Mechanism: stepwise JSON generation with smaller scoped prompts instead of one huge config prompt

- **Section prompt**
  - Function: turn graph and simulation evidence into a predictive report section
  - Mechanism: mandatory tool usage, explicit `Final Answer` boundary, no-heading constraint, quote-first evidence framing, and code-enforced retries when the model mixes tool calls with final content

- **InsightForge sub-query prompt**
  - Function: widen one analytical question into several observable retrieval angles
  - Mechanism: ask for simulation-relevant sub-questions and inject section-level `report_context`

- **Interview selection + question prompts**
  - Function: turn a vague desire for “voices from the simulation” into a concrete interview plan
  - Mechanism: choose relevant agents from profile summaries, then generate short open questions tailored to those roles

- **Interview prefix**
  - Function: force simulated agents to answer directly rather than escaping into tool calls or JSON wrappers
  - Mechanism: prepend an imperative prefix before the prompt reaches the live OASIS environment

- **Chat prompt**
  - Function: answer follow-up questions efficiently after a report exists
  - Mechanism: report-first answering policy, small tool budget, concise style instructions

## Open Questions

- Which prompts are most brittle under longer documents or denser graphs.
- Whether the planning prompt should expose more explicit evidence summaries from the graph instead of only a small fact sample.
- Whether the section prompt’s 3-tool rule is always beneficial, or sometimes over-constrains short sections.
- Whether interview answers stay high quality when the prefix suppresses tool use but the persona memory is large.
- Whether the markdown-heavy `InterviewResult.to_text()` format makes it harder for the section prompt to stay heading-free and stylistically clean.
- Whether unused-tool hints actually increase evidence diversity in real runs or just create ritualistic extra tool calls.

## Next Action

- Compare live outputs from `agent_log.jsonl` against the prompt contracts to see:
  - which tool descriptions the model actually follows
  - whether interviews are chosen only when the simulation is ready
  - where heading drift, tool-call formatting drift, or weak close-out behavior still appears
