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
