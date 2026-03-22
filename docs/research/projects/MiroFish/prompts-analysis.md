---
project: MiroFish
source_path: examples/MiroFish
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: separate prompt-loop resumability from report-selection resumability, because restart can preserve prompt outputs while still choosing the wrong report artifact for later chat
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
- The section prompt is only the static base layer. The runtime loop appends more control text after each turn:
  - observation wrapper
  - insufficient-tool warning
  - tool-limit warning
  - force-final message
  - format-error repair prompt when tool call and `Final Answer:` appear together
- The section loop feeds the model markdown-heavy tool observations, including structured interview reports, and then asks the model to turn them back into heading-free section prose.
- The code keeps nudging prompt behavior toward tool diversity by showing unused-tool hints after each observation.
- The report chat prompt is structurally different from the section prompt: it is report-first, concise, and tool-optional rather than tool-mandatory.
- A validated live run now shows that the section prompt contract is not hypothetical. All 3 sections actually satisfied the “at least 3 tools” requirement before closing.
- In that live run, the first 3 tool calls were highly consistent across sections:
  - `panorama_search`
  - `insight_forge`
  - `interview_agents`
- Section 2 then added `quick_search` as a fourth step after the interview result came back empty.
- The live interview results looked structurally rich but informationally poor inside the report loop:
  - agent selection worked
  - question generation worked
  - interview summaries were produced
  - the report-facing per-agent answers were still shown as empty on both platforms
- Follow-up inspection now shows that the single-platform batch response actually carried substantive answers, but the report tool discarded them while parsing.
- Section 2 exposed a prompt/runtime mismatch: the model closed with prose that lacked the expected `Final Answer:` prefix, yet the code accepted it as final content because there was no further tool call.
- The final section markdowns did not contain explicit interview transcripts or interview-specific quoted replies, which means the prompt/runtime stack currently treats empty interview summaries as optional background rather than must-cite evidence.

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
- In live use, the section prompt plus orchestration code produce a repeatable “research ritual” more than open-ended tool choice. The model repeatedly follows the same broad -> deep -> interview rhythm even when the decoded report-layer interview evidence looks weak.
- The current prompt stack successfully preserves boundedness and closure, but it does not yet encode a strong notion of “low-yield tool, switch strategy”.
- The prompt stack is probably less responsible for the apparent interview weakness than first thought. At least one major source of weakness comes from the adapter layer losing valid responses before they reach the section prompt.
- Prompt behavior in `MiroFish` is best understood in three layers:
  - static templates
  - loop-time injected control messages
  - post-generation cleaners that normalize heading drift

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
  - In the validated run, they effectively induced a stable default order of breadth first, then decomposition, then live interview.

- **Tool-meta prompts**
  - `InsightForge` asks the model to decompose one analytical question into up to 5 smaller observable sub-questions.
  - Interview selection asks the model to choose a diverse and relevant subset of agents.
  - Interview-question generation asks for 3-5 short open questions.
  - Interview-summary synthesis asks for consensus, disagreement, and quotable lines.

- **Chat and interview prompts**
  - Keep responses concise.
  - Limit tool use.
  - In interview mode, suppress tool use completely.

## Prompt Structure Deep Dive

- `PLAN_SYSTEM_PROMPT`
  - role and worldview
  - output schema
  - section-count limit
- `PLAN_USER_PROMPT_TEMPLATE`
  - simulation requirement
  - graph statistics
  - small fact sample
- `SECTION_SYSTEM_PROMPT_TEMPLATE`
  - worldview reset
  - evidence rules
  - translation rule
  - formatting bans
  - tool catalog
  - turn protocol
  - content constraints
- `SECTION_USER_PROMPT_TEMPLATE`
  - truncated previous sections
  - current section assignment
  - first-turn operating instructions
- loop-injected prompts
  - observations
  - repair messages
  - budget reminders
  - force-close instruction
- `CHAT_SYSTEM_PROMPT_TEMPLATE`
  - report-first answer policy
  - concise style
  - optional retrieval

## Core Prompt Elements

- worldview framing
  - future prediction
  - god-view over the simulation
- evidence rule
  - no unstated knowledge
  - use simulated events and agent behavior
- language normalization
  - translate mixed-language evidence into Chinese
- formatting control
  - no headings
  - use bold and lists instead
- loop protocol
  - one tool or final answer, never both
- closure rule
  - `Final Answer:` for sections
- retrieval prior
  - mix tools, avoid one-tool monoculture
- chat compression
  - short direct answers, tools only when needed

## Prompt Pattern Notes

- The same notion appears repeatedly as future prediction rather than present-tense analysis.
- The report layer actively tells the model what not to do, which is a sign that format adherence matters more than open-ended creativity.
- The interview prefix is a practical guardrail: it reduces the chance that the agent will try to invoke tools during a manual response channel.
- The strongest prompt pattern in this repo is split declarative goal plus procedural constraints plus code enforcement.
- The prompt system is modular by function rather than by one giant universal prompt.
- `ReportAgent` uses prompt structure and code together to create a bounded research dialect:
  - prompt says “use tools, no headings, evidence first”
  - code says “one tool per turn, at least 3 tools, stop after 5”
- Live evidence suggests one more layer to that dialect:
  - prompt says interviews make the section more vivid and grounded
  - code treats any successful tool return as progress
  - together they can normalize an interview call even when the actual conversational evidence is empty
- A useful distinction in this repo is:
  - prompt structure explains what the model sees at rest
  - loop control explains what the model is forced to see over time

## Prompt Loop And Resumability

- Section prompts operate as local control surfaces, not as durable checkpoints.
- The effective section-writing contract is distributed across:
  - static system prompt
  - static user prompt
  - tool observation wrapper
  - insufficient-tool warning
  - tool-limit warning
  - mixed tool/final-answer conflict repair
  - force-final fallback
- Those runtime-injected control messages are part of the true prompt mechanism even though they are not stored as standalone prompt files.
- They are also not persisted as a resumable loop snapshot.
- What survives a crash or backend restart is therefore:
  - completed section artifacts
  - structured logs of prior prompt/tool turns
  - not the exact in-flight `messages` stack that would have generated the next turn
- The chat surface follows a different resumability model:
  - it rebuilds a fresh prompt from saved report markdown plus client-supplied recent turns
  - it does not attempt to revive any prior section-writing prompt state
- The restart probe adds a practical caveat:
  - even this reconstructive chat path depends on selecting the intended saved report
  - with multiple reports under one `simulation_id`, chat may rebuild from the wrong artifact and collapse to "暂无报告"
- This means prompt reconstruction quality is now bottlenecked by artifact selection policy as much as by prompt design itself.

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
  - Live effect: in the validated run, this mechanism repeatedly yielded `panorama_search -> insight_forge -> interview_agents`, and once the interview result was weak, the loop optionally reached for `quick_search`

- **InsightForge sub-query prompt**
  - Function: widen one analytical question into several observable retrieval angles
  - Mechanism: ask for simulation-relevant sub-questions and inject section-level `report_context`

- **Interview selection + question prompts**
  - Function: turn a vague desire for “voices from the simulation” into a concrete interview plan
  - Mechanism: choose relevant agents from profile summaries, then generate short open questions tailored to those roles
  - Live effect: selection and question generation worked, and the underlying single-platform answer stream was non-empty, but the report adapter failed to decode that stream correctly

- **Loop repair prompts**
  - Function: keep the section loop inside a valid state machine
  - Mechanism: reject mixed tool/final outputs, reject early closure, reject over-budget tool use, and force a final answer when the loop runs out of room

- **Interview prefix**
  - Function: force simulated agents to answer directly rather than escaping into tool calls or JSON wrappers
  - Mechanism: prepend an imperative prefix before the prompt reaches the live OASIS environment

- **Chat prompt**
  - Function: answer follow-up questions efficiently after a report exists
  - Mechanism: report-first answering policy, small tool budget, concise style instructions
  - Restart implication: the chat mechanism is reconstructive rather than resumptive, because it reloads saved report artifacts and trusts the client to resend recent history

## Open Questions

- Which prompts are most brittle under longer documents or denser graphs.
- Whether the planning prompt should expose more explicit evidence summaries from the graph instead of only a small fact sample.
- Whether the section prompt’s 3-tool rule is always beneficial, or sometimes over-constrains short sections.
- Whether interview answers stay high quality when the prefix suppresses tool use but the persona memory is large.
- Whether the markdown-heavy `InterviewResult.to_text()` format makes it harder for the section prompt to stay heading-free and stylistically clean.
- Whether unused-tool hints actually increase evidence diversity in real runs or just create ritualistic extra tool calls.
- Whether the prompt should tell the model to de-prioritize `interview_agents` after one empty interview result in the same report, or whether that policy question should wait until the parsing bug is removed.
- Whether the `Final Answer:` requirement should be enforced more strictly, since section 2 closed successfully without it.
- Whether the planning prompt is too detached from later section loops because it only sees a small fact sample instead of a richer evidence map.
- Whether the project should persist a lightweight prompt-loop checkpoint for unfinished sections, or continue relying on artifact replay plus fresh prompt reconstruction.
- Whether post-restart chat should accept `report_id` explicitly, so prompt reconstruction is anchored to one artifact rather than to an ambiguous simulation-scoped lookup.

## Next Action

- Compare live outputs from `agent_log.jsonl` against the prompt contracts to see:
  - whether truly decoded interview evidence changes later tool choice
  - whether the section prompt can be adjusted to prefer `quick_search` or graph-only closure when interviews are low-yield
  - whether final-answer formatting drift is common enough to deserve an explicit rule update
  - which behaviors belong to prompt design and which actually belong to loop-control code
