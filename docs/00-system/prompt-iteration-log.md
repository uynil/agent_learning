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
