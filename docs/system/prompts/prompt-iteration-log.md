# Prompt Iteration Log

This file records how the research prompts and adjacent workflow rules evolve over time.

## Version: v1

- Date: 2026-03-21
- Scope: initial research operating system
- Status: active

### What exists in v1

- one master prompt for routing the research mode
- four analysis prompts for quick scan, architecture, orchestration, and prompt audit
- one doc refinement prompt
- fixed output categories for evidence and uncertainty

### Why this version exists

The first goal is not maximum sophistication. The first goal is to create a stable, reusable study loop that can survive multiple projects and multiple tools.

### Known risks

- the prompts may still be too broad for very large projects
- comparison prompts are not separated yet by topic depth
- project-specific constraints may require a smaller prompt variant later
- prompt changes still need stable smoke cases for regression checks

## Version: v1.1

- Date: 2026-03-21
- Scope: review-driven refinement of the docs workflow and prompt operating system
- Status: active

### What changed

- reduced document overlap by splitting responsibilities more clearly:
  - `README.md` is now the human entry page
  - `learning-workflow.md` is the workflow source of truth
  - `AGENTS.md` is the execution rules file
- changed project note ownership so each project uses `README.md` as its canonical entry page
- added `session-close-checklist.md`
- added `prompt-smoke-cases.md`
- added `research-index.md`
- added stronger rules for blocked and low-confidence research
- added promotion thresholds before lessons can move into `AGENTS.md`

### What improved

- the system is less likely to drift because rules, workflow, and prompts now have clearer owners
- project notes are less likely to split across multiple overview files
- prompt changes now have a path toward repeatable regression checks
- failed research sessions can leave behind resumable output instead of disappearing
- long-term maintenance should get lighter because high-frequency and low-frequency updates are now separated

### What got worse

- the system now has a few more supporting docs to maintain
- closing a session is more explicit, which adds a little friction in the short term
- the workflow depends more on discipline around checklists and thresholds instead of "just remember it"

### Evidence from actual use

- the first engineering review found that the workflow had been repeated across `README.md`, `AGENTS.md`, and `learning-workflow.md`, which created immediate drift risk before any real project study had even started
- project placeholders were already encouraging a second overview file name (`project-overview.md`), which would have caused split ownership later
- the review exposed a real blind spot: prompt quality was being discussed without any fixed smoke cases
- the review also exposed that blocked or low-confidence sessions were not formally captured, which would have caused repeated failed exploration

### Modification learnings

- for a learning system, document ownership matters as much as prompt quality
- if a rule appears in three files, it will eventually become three different rules
- a project research workspace should optimize for resumability, not just for "happy path" study sessions
- lightweight structure added after review is better than a large speculative structure added before first real use

### Testing learnings

- a docs-and-workflow system should be tested like an operational process, not like a software library
- the most useful tests here are:
  - session-close checklist validation
  - prompt smoke cases
  - failure-mode review
  - explicit high-frequency vs low-frequency update boundaries
- manual smoke cases are the right first step; automation would be premature before the cases stabilize
- silent failure is the main risk:
  - workflow drift
  - premature rule promotion
  - lost blocked research
  - split project entry points

### Tooling notes

- the review logging scripts needed explicit `SLUG` and `BRANCH` environment variables in this repo state
- the dashboard output was weak because the repository still has no commits on `master`
- this means process checks should not assume a mature git history exists

### Next hypothesis

- after the first 2-3 real project studies, the strongest next improvement will likely be refining the smoke cases and tightening the `AGENTS.md` promotion criteria
- once real project notes exist, the topic index should reveal whether current comparison dimensions are enough or still too abstract

## Version: v1.2

- Date: 2026-03-21
- Scope: extending the research system for prompt structure, context management, and deep research loops
- Status: active

### What changed

- expanded prompt research from reliability-only concerns into:
  - prompt function
  - prompt structure
  - core prompt elements
  - function-to-mechanism mapping
  - prompt composition
  - prompt-to-code coupling
  - reliability and failure modes
- added `Context Management Audit` and `Deep Research Mode` to the workflow
- added new prompts:
  - `Context Management Prompt`
  - `Deep Research Prompt`
  - `Question Rewrite Prompt`
  - `Round Summary Prompt`
- added new methodology docs:
  - `how-to-analyze-context-management.md`
  - `how-to-analyze-deep-research-mode.md`
- updated templates, checklist, research index, and smoke cases to match the new themes

### What improved

- the system can now study how other agents handle long context, memory, retrieval, and resumable research loops
- deep research is now treated as an explicit operating mode instead of an informal "keep going" behavior
- prompt analysis is less likely to stay shallow because function, structure, core elements, and function-to-mechanism mapping are now first-class questions
- multi-round sessions now have a clearer requirement for rewritten questions and resumable summaries

### What got worse

- the prompt system is larger and slightly harder to memorize
- there are now more optional research paths, which raises the importance of choosing one primary theme per round
- the workflow relies more on disciplined round summaries when a session goes deep

### Evidence from actual use

- the design discussion exposed that "prompt reliability" was too narrow and would miss prompt structure, core elements, and composition patterns
- the follow-up discussion exposed a second blind spot: even with better structural analysis, the system still needed an explicit way to explain how a prompt actually produces its intended behavior
- the request to study "how other agents implement deep research" showed that the old workflow lacked explicit support for query rewrite and multi-round investigation
- the earlier workflow also did not force a resumable round summary, which would make deep sessions harder to continue later

### Modification learnings

- prompt research needs both static analysis and systems analysis:
  - static analysis for structure and core elements
  - systems analysis for composition, coupling, and runtime behavior
- prompt analysis also needs function-to-mechanism mapping:
  - what the prompt is trying to achieve
  - which prompt element likely carries that effect
  - how that element cooperates with orchestration code
- context management deserves its own lens and should not be hidden inside generic orchestration notes
- deep research mode works better as a loop with explicit question rewriting than as an unbounded long-form analysis prompt

### Testing learnings

- smoke cases must cover more than quick scan and prompt reliability; they also need:
  - context-heavy projects
  - deep-research projects
  - resumable round summaries
- session-close checks need a deep-research branch, otherwise multi-round work will drift or become hard to resume
- template changes matter for testing because the workflow fails silently when new prompt capabilities have nowhere structured to land

### Next hypothesis

- after the first real deep-research project, the next likely refinement will be tightening the stop conditions for when to end a round versus continue one more pass
- prompt audit may later need a smaller "prompt structure only" variant if the full prompt audit becomes too heavy for quick sessions

## Version: v1.3

- Date: 2026-03-21
- Scope: first real-project prompt-audit learning from `autoresearch`
- Status: active

### What changed

- refined the working interpretation of prompt analysis:
  - inspect command-level instructions as prompt mechanisms, not only natural-language framing
  - explicitly look for context-throttling instructions such as log redirection, grep extraction, and file-based resumability

### What improved

- the prompt-audit lens is now better at catching prompts that shape agent behavior through shell workflow and external artifacts
- orchestration analysis and prompt analysis now connect more naturally in projects where the control plane lives mostly in prompt text

### What got worse

- prompt analysis now has one more thing to remember during quick scans
- the boundary between prompt design and orchestration design needs more deliberate judgment in minimal systems

### Evidence from actual use

- the `autoresearch` study showed a prompt that manages context not by summarization prompts, but by telling the agent to redirect training output to `run.log`, grep only key metrics, and keep experiment memory in `results.tsv`
- this means some of the most important prompt mechanisms may be ordinary shell instructions that reshape context flow and state persistence outside the model

### Modification learnings

- prompt mechanisms can be operational, not just rhetorical
- a good prompt audit should ask:
  - which commands keep context small
  - which artifacts carry memory across rounds
  - which instructions implicitly create the orchestration state machine

### Testing learnings

- prompt smoke cases should eventually include at least one project where the prompt controls logging, parsing, and rollback through shell commands rather than through structured JSON schemas

### Next hypothesis

- future prompt-analysis docs may need an explicit checkpoint for "context shaping via tools and files" so this pattern does not get lost in larger projects

## Version: v1.4

- Date: 2026-03-21
- Scope: prompt-analysis learning from `self-evolving-agent`
- Status: active

### What changed

- expanded the prompt-analysis lens to handle multi-document protocol stacks:
  - top-level skill contract
  - coordinator document
  - step-local protocol modules
  - asset templates
  - hook reminders

### What improved

- the workflow is now better suited for projects where the “prompt system” is really a layered operating manual rather than a single prompt file
- prompt analysis can now better explain authority boundaries:
  which file sets philosophy, which file schedules steps, and which file defines local output contracts

### What got worse

- prompt analysis is a little less lightweight because it now has to distinguish prompt layers inside skill systems, not just inside LLM request builders

### Evidence from actual use

- `self-evolving-agent` does not center its behavior in one prompt string
- its behavior is split across `SKILL.md`, `system/coordinator.md`, multiple protocol modules, persistent ledger templates, and a bootstrap hook reminder
- analyzing only one of those files would miss the real control logic

### Modification learnings

- some projects use prompts as a governance stack, not just as task prompts
- hook reminders can function as compressed retrieval cues for a much larger protocol system
- output templates in asset files may be part of the prompt system when they shape how the agent records and reuses state

### Testing learnings

- prompt-oriented research should include at least one smoke case where the prompt layer is distributed across multiple documents and a hook, not only across code-built prompt strings

### Next hypothesis

- future prompt-analysis docs may need an explicit “prompt stack map” section for skill-based systems with layered protocol documents

## Version: v1.4

- Date: 2026-03-21
- Scope: first real-project learning from a documentation-first local retrieval skill (`rag-skill`)
- Status: active

### What changed

- refined the prompt-analysis lens to explicitly capture two mechanisms in file-backed retrieval systems:
  - manually curated index docs that guide where the agent should search
  - mandatory pre-read reference docs that teach the agent how to use tools before touching certain file types
- tightened the classification boundary between:
  - retrieval systems with actual retrieval infrastructure
  - prompt-governed retrieval workflows over a file system

### What improved

- the research system is now better at describing projects whose "RAG" behavior lives mostly in docs, skill instructions, and shell/tool workflow rather than in embeddings or vector search code
- prompt analysis can now explain why some supporting docs are not just background reading, but active control-plane components for tool use

### What got worse

- classifying a project as "RAG" now needs slightly more care because the label may describe the user experience more than the mechanism
- prompt audits now have one more subtle pattern to check, which adds a bit of judgment overhead

### Evidence from actual use

- the `rag-skill` study found a repository with no visible embedding or vector-store layer, but with a clear retrieval workflow driven by:
  - `data_structure.md` directory indexes
  - mandatory file-type reference docs
  - iterative `grep` and local-read behavior
- this means a project can present as "RAG" while its practical mechanism is really a prompt-enforced file-navigation protocol

### Modification learnings

- some retrieval systems are better analyzed as workflow contracts than as retrieval algorithms
- "read this reference before using the tool" is a real prompt mechanism, not just documentation hygiene
- manually maintained index docs deserve explicit treatment as retrieval metadata when they control search order and scope
- local retrieval analysis should distinguish metadata provenance:
  - hand-maintained index docs
  - generated knowledge maps
  because the two shapes create very different maintenance and failure profiles

### Testing learnings

- when studying local knowledge-base systems, compare documentation claims against the actual checked-out files because manual indexes drift
- runtime validation for this class of system should test not only answer quality, but also whether the agent obeys prerequisite tool-learning steps before handling PDFs or spreadsheets
- runtime validation should also probe tool availability up front, because a well-documented PDF or spreadsheet path may still fail if the recommended local tools are missing
- tool probing should happen in the target project working directory, because command resolution like `python` vs `python3` can change the available environment and therefore the observed behavior
- for app-backed retrieval systems, runtime validation should be staged:
  - app boot
  - metadata endpoints
  - retrieval endpoints
  - real provider-backed answer generation
  so placeholder credentials or provider errors do not get mistaken for backend or retrieval failures
- when validating tag-based ReAct flows, parse markers with message-role awareness rather than naive global string matching, because system-prompt examples may contain the same tags and can fool a test harness into skipping the real tool step
- for systems that support both native tool calls and ReAct fallback, run the same query through both modes because runtime cleanliness can diverge sharply even when the retrieval contract is shared
- distinguish raw transport behavior from rendered UX behavior, because a frontend may clean or hide protocol markers that still exist in the backend stream
- when a repository includes both markdown prompt docs and code-embedded prompt strings, verify which one is actually executed before treating the markdown file as a source of truth
- when auditing in-app config editors, trace whether a save updates one local settings handle or the actual shared runtime state, because module-level singletons and imported settings objects can silently preserve stale configuration
- in retrieval comparisons, separate substrate shape from retrieval-control locus, because two systems can both be "local RAG" while one lets the model choose scope at runtime and the other encodes scope choice in retrieval modes and storage structure
- compare fallback carriers, not just primary retrieval paths: what still produces context when the preferred branch weakens often explains a system's real resilience profile better than the headline architecture does
- once a project-level observation has direct user or security impact, a precise code path, and little dependence on product interpretation, promote it from a research note to a review-grade candidate instead of leaving it as vague architectural drift

### Next hypothesis

- after one or two more local knowledge-base projects, the workflow may need a dedicated checklist item for "manual index plus prerequisite reference" patterns if they keep recurring

## Version: v1.5

- Date: 2026-03-21
- Scope: live-validation prompt bridging for toolful local skills on a `chat/completions`-only provider
- Status: active

### What changed

- refined the live-validation prompt lens to distinguish:
  - protocol document injection
  - runtime contract injection
- treated "no tools are available here" as a first-class prompt bridge when a local skill is moved from a toolful runtime into a plain chat-completions interface

### What improved

- the research system can now explain why the same skill documents can produce:
  - pseudo tool-call output in one manual live run
  - a clean benchmark-grade user-facing answer in another
- manual live validation is now easier to interpret because prompt failures can be separated from provider failures

### What got worse

- cross-provider validation got a little less apples-to-apples, because some providers now need an extra bridging clause to reproduce the runtime assumptions that Codex would have provided implicitly

### Evidence from actual use

- in `self-evolving-agent`, a direct `chat/completions` run with injected `SKILL.md` and `system/coordinator.md` produced pseudo `tool_call` blocks instead of a final diagnosis
- a second run against the same provider, same model, and same scenario passed once the prompt also enforced:
  - final answer only
  - no benchmark internals
  - no tools available in the environment
- the ad hoc judge then scored all four required criteria at `2/2`

### Modification learnings

- when porting a local skill from a toolful agent runtime to a plain chat API, injecting the skill docs is not enough
- the prompt also has to restate the execution environment contract, or the model may continue reasoning as though tools are available
- runtime assumptions are part of the effective prompt, even when they were implicit in the original environment

### Testing learnings

- for manual live validation on compatibility providers, run at least two probes:
  - raw protocol-doc injection
  - protocol-doc injection plus explicit runtime contract
- judge the two probes separately so tool-runtime mismatch does not get misclassified as skill failure

### Next hypothesis

- if more projects show the same pattern, benchmark runners may need first-class provider adapters that translate tool/runtime assumptions into prompt-level contracts for `chat/completions` paths

## Version: v1.6

- Date: 2026-03-21
- Scope: promoting a manual compatibility-provider prompt bridge into a native benchmark-runner path
- Status: active

### What changed

- moved the `self-evolving-agent` `chat/completions` bridge out of an ad hoc manual probe and into the repository's benchmark runner design
- kept the bridge explicit:
  - inject protocol documents
  - restate the no-tools runtime contract
  - restate the judge JSON contract

### What improved

- compatibility-provider validation is now stronger because the positive result can be reproduced through the repository's own report flow instead of only through one-off scripts
- prompt research now has a clearer boundary between:
  - "manual probing discovered a workable bridge"
  - "the system actually encodes that bridge as a reusable mechanism"

### What got worse

- benchmark infrastructure is more complex because runner backends now have to carry environment-specific prompt assumptions, not just model and URL configuration

### Evidence from actual use

- in `self-evolving-agent`, four new runner tests passed after adding:
  - timeout-path regression coverage
  - tool-less candidate prompt coverage
  - skill-context chat backend coverage
  - judge JSON contract coverage
- the native benchmark runner then passed the first scenario through a `chat/completions` backend with score ratio `1.00`

### Modification learnings

- when a manual prompt bridge proves necessary and stable, the next useful step is often to encode it in the runner rather than keep repeating it in research notes
- provider compatibility is not only an API-surface question; it can require translating runtime assumptions into prompt-level adapters

### Testing learnings

- after adding a compatibility backend, do not stop at mocked tests
- immediately run at least one real scenario through the native report flow so the bridge is validated at the orchestration layer, not just at the request-construction layer

### Next hypothesis

- the remaining benchmark scenarios may reveal whether the current bridge is generally sufficient or whether some skill systems need per-scenario or per-role prompt adapters

## Version: v1.7

- Date: 2026-03-21
- Scope: separating declared RAG product surface from effective runtime surface
- Status: active

### What changed

- sharpened the research lens so project analysis now explicitly compares:
  - README and spec claims
  - visible UI or CLI controls
  - actual runtime entrypoints and effective config bindings

### What improved

- large RAG repositories are easier to classify without over-crediting their apparent feature breadth
- research notes can now distinguish a genuinely runnable surface from a broad but partially aspirational architecture story

### What got worse

- analysis is a little slower because it is no longer enough to read the main README and a few core modules
- UI-heavy and spec-heavy repositories now require one more pass to identify where the real launch boundary actually is

### Evidence from actual use

- in `deep-rag`, the README and frontend state imply configurable provider/model/path behavior, but the effective runtime surface is narrower:
  the documented `_PATH` env vars drift from the real settings model, the config panel hardcodes localhost, and the frontend has no visible provider/model selector
- in `MODULAR-RAG-MCP-SERVER`, the README and DEV_SPEC present a broad end-to-end modular RAG plus MCP platform, but the packaged root entrypoint still logs that the MCP server "will be implemented in Phase E" even though dedicated MCP modules already exist deeper in `src/`

### Modification learnings

- repository breadth does not equal runnable completeness
- for spec-driven AI projects, the teaching scaffold may be part of the architecture, but it should not be mistaken for live behavior
- the effective product surface often lives at the entrypoint and config layer, not in the most impressive subtree

### Testing learnings

- when analyzing RAG systems with WebUI, MCP, or server packaging, inspect the actual launch target before trusting the README's runtime story
- if the project is not locally checked out, record that boundary explicitly and keep confidence low even when the remote repository looks rich

### Next hypothesis

- more RAG projects will likely show the same gap between declared modularity and effective runtime surface, which may justify a dedicated comparison pattern for "spec-heavy vs runtime-proven" systems

## Version: v1.8

- Date: 2026-03-22
- Scope: separating default launch paths, real module entrypoints, and environment health
- Status: active

### What changed

- refined the runtime-validation lens for modular repositories so analysis now checks three different questions instead of one:
  - what the packaged or documented default entrypoint does
  - what deeper module entrypoints actually implement
  - whether the current workspace environment can execute those deeper paths

### What improved

- project notes can now distinguish "placeholder root launcher" from "missing implementation" more accurately
- runtime confidence is less likely to be inflated by the existence of deep source trees or by broad test directories
- modular systems are easier to classify without collapsing packaging drift and dependency drift into the same diagnosis

### What got worse

- first-pass analysis takes a little longer because a quick source read is no longer enough
- research notes may contain more nuanced partial states instead of a simple runnable / not-runnable judgment

### Evidence from actual use

- in local analysis of `MODULAR-RAG-MCP-SERVER`, the root [`main.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/main.py) is still a placeholder launcher and the packaged script still points there, but [`src/mcp_server/server.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/src/mcp_server/server.py) contains a real MCP stdio server
- that same local pass also showed a second, separate boundary:
  the deeper MCP path still fails in the current workspace because the declared `mcp` dependency is not installed
- the same project also showed a third kind of drift:
  default config values in [`config/settings.yaml`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/config/settings.yaml) no longer match some Azure-oriented expectations in [`tests/integration/test_ingestion_pipeline.py`](/Users/uynil/projects/agents/examples/MODULAR-RAG-MCP-SERVER/tests/integration/test_ingestion_pipeline.py)

### Modification learnings

- "real code exists" is a different claim from "default launch path reaches that code"
- "default launch path reaches that code" is a different claim from "current environment can execute it"
- for modular AI systems, packaging drift, config drift, and dependency drift should be recorded separately because they imply different next actions

### Testing learnings

- for first-pass runtime checks, try both the packaged root entrypoint and the deeper module entrypoint when the source tree suggests they differ
- when tests cannot be executed because the test runner is missing, record that as an environment boundary rather than silently downgrading the codebase itself
- when config files and tests disagree, treat that as a runtime-health signal even before any full integration run

### Next hypothesis

- future comparison docs may benefit from a dedicated pattern for repositories that are "implementation-rich but bootstrap-fragile," especially in modular RAG and MCP projects

## Version: v1.9

- Date: 2026-03-22
- Scope: validating retrieval systems with minimal dependency bootstraps and path-specific fakes
- Status: active

### What changed

- refined the runtime-validation workflow for RAG projects so the first executable proof no longer depends on full provider bring-up
- the new pattern is:
  - install the smallest dependency slice that unlocks the target path
  - validate the real entrypoint when possible
  - use path-specific fake stores or fake search components only for the deeper stage that would otherwise be blocked by external providers

### What improved

- research can now produce real execution evidence even when full OpenAI, Chroma, or server stacks are not configured
- the notes can distinguish protocol health, retrieval-path health, and provider-health instead of collapsing them into one binary result
- this makes partial validation much more informative for modular RAG repositories

### What got worse

- validation output is more nuanced and takes a little more explanation
- some runs now prove a subsystem path rather than an end-to-end product flow, so the confidence boundary must be written carefully

### Evidence from actual use

- in `MODULAR-RAG-MCP-SERVER`, installing only `mcp` was not enough; the real server path also required `pyyaml` before stdio `initialize` and `tools/list` would work
- once those minimal deps were present, the real MCP server path could be validated independently of Chroma or provider SDKs
- after adding `jieba`, a real stdio `tools/call query_knowledge_hub` still returned a valid MCP tool result, but surfaced the next blocker cleanly as missing `chromadb`
- for `query_knowledge_hub`, a local fake `HybridSearch` plus a no-op initializer let the async tool execution path run without pretending the full storage/provider stack was healthy
- in `LightRAG`, a fake chunk vector store was enough to run `naive_query(...)`, while fake graph and vector stores were enough to run `_perform_kg_search(...)` in `mix` mode and prove that KG and vector-chunk branches both participate
- a real `LightRAG` object insert-and-query cycle then confirmed that `mix` can still succeed with chunk retrieval even when dummy extraction yields zero entities and zero relations

### Modification learnings

- for modular retrieval systems, "what is the smallest dependency slice that proves this layer is alive?" is a very useful research question
- path-specific fakes are acceptable when they are used to isolate one blocked layer and the note explicitly says which layer is still unvalidated
- lower-level retrieval-path execution can still be strong evidence, as long as it is not misrepresented as full end-to-end product validation
- a real tool-level error response over the wire is stronger evidence than a local exception, because it proves the protocol surface is handling backend failure correctly

### Testing learnings

- verify protocol layers with the real wire path first when possible
- verify retrieval composition with fake stores when real corpora or providers are the only blocker
- record exactly which parts are real:
  - real entrypoint
  - real protocol
  - real retrieval code
  - fake store or fake provider boundary

### Next hypothesis

- future comparison docs may benefit from a reusable "validation ladder" for RAG systems:
  import health, protocol health, retrieval-path health, corpus-backed health, and full provider-backed health

## Version: v1.10

- Date: 2026-03-22
- Scope: finishing static analysis after partial runtime validation for multi-layer RAG platforms
- Status: active

### What changed

- started splitting deeper static conclusions into optional per-project `architecture.md` notes once the README already had enough status, evidence, and classification context
- treated the remaining work for `MODULAR-RAG-MCP-SERVER` and `LightRAG` as comparison-ready follow-up rather than requiring more project-local file spelunking first

### What improved

- project READMEs can stay focused on current evidence and confidence, while deeper structural findings get a stable place to live
- analysis completion is easier to call honestly:
  once architecture, state model, and runtime boundaries are covered, the next step can be comparison rather than more source gathering

### What got worse

- the project folder can gain one more document when the architecture is rich enough
- there is a small new judgment call about when a deeper note is warranted versus when the README is still sufficient

### Evidence from actual use

- the deeper `MODULAR-RAG-MCP-SERVER` pass surfaced a clear three-part architecture:
  typed pipeline contracts, file-backed retrieval state, and an MCP surface whose packaging still lags the real server implementation
- the deeper `LightRAG` pass surfaced a different shape:
  a single large control-plane dataclass, a shared-storage substrate for workspace and lock semantics, and mode-based retrieval dispatch that cleanly separates KG, chunk-only, and bypass paths

### Modification learnings

- after a retrieval project has some runtime evidence, the highest-value static question is often "where does state actually live?" rather than "what other files exist?"
- architecture notes become justified when the README would otherwise have to carry both status reporting and a second layer of structural explanation

### Testing learnings

- for retrieval platforms with many modes, inspect the central query dispatchers before reading outer route handlers, because they usually reveal the real branch boundaries faster
- for systems with shared-storage or trace layers, those files often explain runtime behavior better than the top-level README or server entrypoint

### Next hypothesis

- future comparison docs may benefit from separating three axes explicitly:
  retrieval topology, runtime state model, and serving-surface complexity

## Version: v1.11

- Date: 2026-03-22
- Scope: comparison-layer synthesis for local-knowledge RAG systems
- Status: active

### What changed

- promoted a new comparison habit for RAG projects:
  separate retrieval topology from state management instead of trying to explain both in one project summary
- added a dedicated retrieval-topology comparison layer once `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` had enough evidence

### What improved

- cross-project conclusions are easier to reuse because the main RAG differences are now expressed as stable patterns instead of ad hoc project summaries
- local-knowledge systems are easier to classify without flattening them all into one vague "RAG" bucket

### What got worse

- comparison work now adds one more document when a family of systems becomes rich enough
- there is a little more judgment involved in deciding whether a difference belongs under topology, state, or serving surface

### Evidence from actual use

- `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` looked superficially comparable as local RAG systems, but the strongest reusable difference was not "which one is better"
- the strongest reusable difference was architectural shape:
  knowledge-map navigation runtime, modular hybrid chunk-retrieval service, and graph-plus-vector substrate platform

### Modification learnings

- once project-level analysis is complete, the highest-signal comparison move is often to ask:
  what is the retrieval substrate, where does state live, and how large is the serving surface?
- for RAG systems, retrieval topology and state model are distinct enough that combining them too early makes the comparison less reusable

### Testing learnings

- do not wait for perfect end-to-end validation before starting comparison work if the retrieval core and runtime boundaries are already clear
- partial runtime evidence plus complete static architecture coverage is often enough to compare system shapes honestly

### Next hypothesis

- future RAG comparisons may benefit from a third dedicated document on serving-surface complexity once more API-plus-WebUI-plus-tooling platforms are analyzed

## Version: v1.12

- Date: 2026-03-22
- Scope: separating retrieval topology from serving-surface complexity in RAG comparisons
- Status: active

### What changed

- added a dedicated serving-surface comparison layer for `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG`
- stopped treating "how retrieval works" and "how users/programs touch the system" as one combined comparison question

### What improved

- comparison conclusions are easier to reuse because a project can now be strong in retrieval topology but weak in default serving shape, or vice versa
- app-coupled systems, protocol-first systems, and platform-scale servers are easier to distinguish cleanly

### What got worse

- comparison space is now slightly more fragmented across documents
- there is one more taxonomy choice to keep consistent across future RAG analyses

### Evidence from actual use

- `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` all had meaningful serving layers, but those layers failed in very different ways:
  frontend/backend contract drift, launcher/protocol drift, and platform-surface complexity

### Modification learnings

- for retrieval systems, the serving surface is often an independent design dimension rather than a simple wrapper around the retrieval core
- "best retrieval architecture" and "best default user-facing surface" should not be collapsed into one judgment

### Testing learnings

- runtime validation should name which serving surface was actually tested:
  backend app, MCP wire protocol, object-level library path, WebUI path, or compatibility API
- this makes partial validation much easier to compare honestly across systems with very different exposure layers

### Next hypothesis

- future comparison work may benefit from a fourth reusable axis for "configuration surface complexity" once more multi-provider or multi-backend systems are analyzed

## Version: v1.13

- Date: 2026-03-22
- Scope: deeper comparison of prompts, context management, and loop structure for local-knowledge RAG systems
- Status: active

### What changed

- split the next comparison pass into three separate axes:
  prompt control, retrieval context, and agent loop shape
- stopped treating "agenticity" as a single scalar and instead compared who owns iteration, what gets re-injected, and where prompts actually carry control

### What improved

- `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` are now easier to compare without flattening them into the same kind of RAG
- prompt-heavy systems, prompt-light service systems, and prompt-layered substrate systems can now be described with cleaner language

### What got worse

- the comparison layer now has more documents, so navigation takes a little more discipline
- there is slightly more taxonomy overhead in keeping prompt, context, loop, topology, and serving surface distinct

### Evidence from actual use

- a deeper source pass showed that the three systems use prompts very differently:
  `deep-rag` uses prompts to steer visible retrieval behavior, `MODULAR-RAG-MCP-SERVER` relies much more on code and protocol contracts, and `LightRAG` uses prompts heavily inside internal extraction and answer-building stages
- the same pass also showed that only `deep-rag` has a truly model-directed retrieval loop at query time; the other two are better described as service-call and pipeline-directed systems

### Modification learnings

- for local-knowledge RAG systems, prompts can serve three different control roles:
  user-visible loop steering, protocol-light support, or internal substrate shaping
- context comparison becomes clearer when framed around "what gets re-injected into the next step?" rather than only "what memory exists?"

### Testing learnings

- when comparing loop shapes, inspect the actual break conditions and fallback branches in source before calling something "agentic"
- when comparing prompt behavior, look at both prompt templates and the code that wraps them, because prompt weight can be hidden in task decomposition rather than in long system prompts

### Next hypothesis

- future comparison work may benefit from a dedicated "configuration-surface complexity" doc, especially for systems like `LightRAG` where provider, storage, and mode combinatorics materially shape runtime behavior

## Version: v1.14

- Date: 2026-03-22
- Scope: turning multi-document local-RAG comparison work into a reusable design-selection guide
- Status: active

### What changed

- added a selection-oriented comparison page for local-knowledge RAG systems
- treated the earlier comparison docs as supporting axes and added one page that answers the practical question:
  when should someone choose each pattern?

### What improved

- the comparison layer is now useful not only for analysis, but also for design decisions
- the growing comparison taxonomy is easier to navigate because there is now one decision page that points back to the underlying axes

### What got worse

- there is now a mild risk of repeating conclusions if the selection page and axis pages are not kept aligned

### Evidence from actual use

- once retrieval topology, serving surface, prompt control, context management, and loop structure were separated, the next useful artifact was no longer another axis
- the next useful artifact was a page that recombined them into direct design heuristics for `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG`

### Modification learnings

- comparison layers eventually need a "decision synthesis" page once enough axes exist
- otherwise the research becomes rich but awkward to apply when someone needs to make a real architecture choice

### Testing learnings

- selection pages should not invent new evidence; they should only recombine already-supported conclusions from the lower-level comparison docs
- this keeps the design advice honest and makes later updates easier

### Next hypothesis

- if more local RAG projects are added, this selection page may need to split into separate guides for lightweight file-backed systems, service-oriented retrieval systems, and substrate-heavy retrieval platforms

## Version: v1.15

- Date: 2026-03-22
- Scope: configuration-surface comparison for local-knowledge RAG systems
- Status: active

### What changed

- added a dedicated comparison page for configuration surface complexity
- separated provider/storage/env/runtime-propagation questions from serving-surface and retrieval-topology questions

### What improved

- configuration problems are now easier to discuss precisely instead of being mixed into vague "runtime drift"
- the design-selection layer now has a clearer operational complement:
  not just which architecture to choose, but what kind of config burden comes with it

### What got worse

- the comparison taxonomy is still growing, so there is more surface area to keep aligned

### Evidence from actual use

- `deep-rag` showed that a small config surface can still be risky when env-name drift and partial hot-reload semantics exist
- `MODULAR-RAG-MCP-SERVER` showed that a typed settings model is helpful, but bootstrap drift can still dominate the real user experience
- `LightRAG` showed that a powerful config matrix creates a different class of risk: not drift, but combination complexity

### Modification learnings

- configuration surface deserves its own comparison axis once a project family includes both app-shaped systems and platform-shaped systems
- "how much config exists?" is much less useful than:
  where config lives, how it propagates, and how easy it is to trust

### Testing learnings

- config validation should distinguish three questions:
  are names aligned, do runtime updates propagate, and is the chosen combination actually healthy
- this makes config analysis more actionable than a generic "docs drift" note

### Next hypothesis

- if more platform-scale projects are added, the next useful split may be between "configuration breadth" and "configuration trustworthiness"

## Version: v1.15

- Date: 2026-03-22
- Scope: separating research observations from review-grade findings during static analysis
- Status: active

### What changed

- added an explicit promotion boundary for when a static observation should stop living only as research prose and start being treated as a review-grade candidate
- started ranking candidate findings by severity inside project notes before turning them into formal review output

### What improved

- static analysis rounds can now stay exploratory without losing the ability to surface concrete defects cleanly later
- project pages can keep architecture insight and code-review risk separate, which makes the research docs easier to trust and easier to continue

### What got worse

- the research workflow now asks for one more judgment call:
  whether something is merely notable architecture drift or already concrete enough to be treated as a defect candidate

### Evidence from actual use

- the `deep-rag` pass surfaced several issues that were no longer just design observations:
  unauthenticated `.env` mutation, partial hot reload, single-call-only tool aggregation, and a hardcoded config endpoint
- leaving those mixed into generic research prose made the notes harder to scan and made severity harder to communicate

### Modification learnings

- a static observation is usually ready to become a review-grade candidate when it has:
  a precise code path
  direct user, security, or deployment impact
  and little remaining ambiguity about whether the behavior is intentional
- severity ordering is worth doing inside the research notes first, because it forces the boundary between "interesting" and "important"

### Testing learnings

- when doing static-only rounds, verify candidate findings against exact line-level evidence before promoting them
- keep the final step optional:
  research docs can hold severity-ranked candidates without forcing a full code review pass every time

### Next hypothesis

- future project templates may benefit from a dedicated `Review-Grade Candidates` section so this boundary stays consistent across studies

## Version: v1.16

- Date: 2026-03-22
- Scope: separating prompt, context, loop, and session comparisons for local-knowledge systems
- Status: active

### What changed

- extended the comparison layer so prompt control, context management, agent loop shape, and session management are treated as related but non-identical axes
- added `rag-skill` as a useful comparison anchor for workflow-governed retrieval behavior
- added dedicated pages for local-knowledge context management and session management instead of overloading broader state notes

### What improved

- local-knowledge systems can now be compared more precisely without flattening everything into generic "RAG architecture"
- the research language is clearer about whether continuity lives in docs, a chat client, request traces, or a persistent substrate

### What got worse

- the comparison layer now has more overlap risk if prompt, context, loop, session, and state docs are not kept distinct

### Evidence from actual use

- `rag-skill`, `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` all deal with continuity and control very differently even though they can all be casually described as local-knowledge retrieval systems
- earlier comparison pages captured parts of that story, but they still mixed session continuity into broader state or context language

### Modification learnings

- `state`, `context`, `loop`, and `session` should not be collapsed into one axis
- a lightweight skill protocol like `rag-skill` is a strong contrast case because it shows continuity governed by instructions rather than by runtime objects

### Testing learnings

- when writing comparison pages, check whether each claimed distinction can be tied back to one concrete owner:
  prompt docs
  backend loop
  client replay
  request trace
  or persistent substrate

### Next hypothesis

- future local-agent comparisons may also need a dedicated axis for human steering versus autonomous continuity once more skill-driven systems are added

## Version: v1.17

- Date: 2026-03-22
- Scope: adding a steering-versus-continuity lens to local-knowledge comparisons
- Status: active

### What changed

- added a dedicated comparison axis for `human steering vs autonomous continuity`
- expanded the local-RAG selection guide so it now synthesizes not only retrieval topology, but also continuity ownership and session location

### What improved

- the comparison layer can now distinguish systems that are "agentic" because they keep working within one request from systems that are "continuous" because they preserve substrate state across many operations
- `rag-skill`, `deep-rag`, `MODULAR-RAG-MCP-SERVER`, and `LightRAG` can now be compared without pretending they all solve autonomy in the same way

### What got worse

- there is now one more comparison axis to maintain, and it sits close to session and loop analysis if the page boundaries are not kept clean

### Evidence from actual use

- the previous round showed that `rag-skill` is continuity-heavy but human-steered, `deep-rag` is more autonomous inside a request, `MODULAR-RAG-MCP-SERVER` is conservative and caller-steered, and `LightRAG` is persistent at the substrate layer rather than the chat-loop layer
- those differences mattered for selection, but they were still awkward to express using only prompt, context, loop, and session pages

### Modification learnings

- "who steers?" and "what persists?" are separate questions
- some systems reduce operator burden by automating loop decisions, while others reduce operator burden by preserving substrate readiness; those should not be treated as the same kind of autonomy

### Testing learnings

- when a comparison starts using the word `agentic`, verify whether it refers to:
  bounded in-request continuation
  cross-request persistence
  or reduced human orchestration burden

### Next hypothesis

- future comparison work may need to distinguish `autonomous continuation` from `autonomous planning` once more true research-agent systems are added

## Version: v1.18

- Date: 2026-03-22
- Scope: separating planning from continuation and resume-from-artifacts from resume-from-runtime
- Status: active

### What changed

- added dedicated comparison pages for `autonomous continuation vs planning` and `artifact vs runtime resume`
- added compact local-knowledge reference cards so the comparison layer has a lightweight scan surface in addition to axis pages

### What improved

- local-knowledge systems can now be compared without treating every bounded loop as a planner
- resume analysis is clearer because it now distinguishes artifact-first recovery from runtime-sensitive interactive continuity

### What got worse

- the comparison stack is denser, so navigation quality matters more than before

### Evidence from actual use

- `deep-rag` is meaningfully more autonomous than `rag-skill` inside one request, but it still is not a planning-heavy system
- `rag-skill` and `MODULAR-RAG-MCP-SERVER` both resume well from artifacts, but for very different reasons:
  one is workflow-driven and one is service-driven
- `LightRAG` preserves more capability across time, but much of that continuity comes from substrate health rather than chat-like runtime state

### Modification learnings

- bounded continuation, explicit planning, and persistence-backed continuity are three different properties
- "resumable" should always be qualified:
  resumable from artifacts
  resumable from client replay
  or resumable from a persistent runtime substrate

### Testing learnings

- when comparing autonomy claims, verify whether the system:
  chooses the next step
  decomposes the task
  or simply preserves enough state to continue later

### Next hypothesis

- future comparison work may benefit from one final meta-index that groups pages by question:
  prompts
  context
  loop
  session
  continuity
  and selection

## Iteration Template

Copy this block for future updates:

```text
## Version: vX

- Date:
- Scope:
- Status:

### What changed

-

### What improved

-

### What got worse

-

### Evidence from actual use

-

### Modification learnings

-

### Testing learnings

-

### Next hypothesis

-
```
