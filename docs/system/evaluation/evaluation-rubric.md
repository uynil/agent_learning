# Evaluation Rubric

Use this rubric to score research output, workflow quality, and prompt stability after each study session.

## Scoring Scale

- `1`: weak
- `2`: limited
- `3`: usable
- `4`: strong
- `5`: excellent

## Dimensions

### 1. Factual Grounding

Question: Is the output clearly anchored in repository evidence?

- `1`: mostly generic
- `3`: some evidence, but uneven
- `5`: consistently tied to files, modules, or functions

### 2. Structural Clarity

Question: Is the output organized well enough to archive and revisit?

- `1`: loose and hard to reuse
- `3`: partly structured
- `5`: stable sections and easy to extend

### 3. Explanatory Power

Question: Does the output explain why the design exists, not just what exists?

- `1`: surface description only
- `3`: some tradeoffs explained
- `5`: strong reasoning and clear tradeoffs

### 4. Boundary Awareness

Question: Does the output clearly mark uncertainty and missing information?

- `1`: overconfident
- `3`: some unknowns called out
- `5`: uncertainty is explicit and useful

### 5. Reusability

Question: Can the output feed future comparisons, methods, or prompt improvements?

- `1`: one-off
- `3`: partly reusable
- `5`: easy to compare, extend, and reuse

### 6. Workflow Compliance

Question: Did the session follow the intended workflow and close-out rules?

- `1`: major workflow steps were skipped or the output landed in the wrong place
- `3`: the main path was followed, but some close-out steps were missed
- `5`: the session followed the workflow cleanly and passed the close-out checklist

### 7. Resumability

Question: If the work stopped here, could someone resume without redoing the exploration?

- `1`: little or no recovery context
- `3`: partial trail, but some context would need to be rediscovered
- `5`: clear status, checked files, blocker state, and next action are recorded

### 8. Promotion Discipline

Question: Were stable lessons promoted carefully instead of being pushed into system rules too early?

- `1`: one-off ideas were promoted without enough evidence
- `3`: promotion decisions were somewhat careful, but still subjective
- `5`: promotions followed explicit thresholds and unstable lessons stayed local

## When to Use This Rubric

### After every research session

Always score at least:

- Factual Grounding
- Structural Clarity
- Boundary Awareness
- Workflow Compliance
- Resumability

### After meaningful prompt changes

Also score:

- Explanatory Power
- Reusability
- Promotion Discipline

And re-run the smoke cases in [prompt-smoke-cases.md](./prompt-smoke-cases.md).

## Session-Close Gate

Before calling a session complete, confirm it also passes [session-close-checklist.md](./session-close-checklist.md).

The rubric scores quality. The checklist confirms operational completeness.

## Smoke Case Guidance

When evaluating prompt changes with smoke cases:

- record which smoke cases were run
- note which outputs improved
- note which outputs regressed
- if two or more smoke cases get materially worse, do not promote the prompt change into stable workflow guidance yet

## Session Review Template

```text
- Project:
- Mode:
- Prompt(s):
- Deep or shallow session:
- Was the session blocked or low-confidence?:

Scores:
- Factual Grounding:
- Structural Clarity:
- Explanatory Power:
- Boundary Awareness:
- Reusability:
- Workflow Compliance:
- Resumability:
- Promotion Discipline:

Checklist:
- Session-close checklist passed?:
- Correct project README updated?:
- Prompt iteration log updated?:
- Failure record needed?:
- Failure record written?:

Smoke cases:
- Smoke cases run:
- Regressions found:

Notes:
- What helped:
- What was too vague:
- What drift risk showed up:
- What should change next:
```
