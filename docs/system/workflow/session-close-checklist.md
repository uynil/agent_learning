# Session-Close Checklist

Use this checklist at the end of every research round.

## Always Check

- The correct project `README.md` was updated
- New facts are separated from inferences
- `status`, `last_updated`, and `next_action` are current
- `docs/system/prompts/prompt-iteration-log.md` was updated if prompts were used

## If the Session Went Deep

- `architecture.md`, `llm-orchestration.md`, `prompts-analysis.md`, or `notes.md` were updated as needed
- the project `README.md` still works as the canonical summary page

## If the Session Was Blocked or Low Confidence

- a failure record was left behind
- checked files are listed
- the blocker is named
- the next resume point is explicit

## Promotion Checks

Only update these if the evidence threshold is met:

- `AGENTS.md`
- `docs/research/comparisons/*`
- `docs/system/prompts/prompt-smoke-cases.md`
- `docs/system/workflow/research-index.md`

Before promoting anything, ask:

- Is this a lesson about the learning system itself?
  - yes -> maybe `AGENTS.md`
- Is this a lesson about agent architecture, orchestration, prompts, or project design?
  - yes -> it belongs in `docs/research/projects/*` or `docs/research/comparisons/*`

## Final Question

If you came back to this repository in two weeks, would you know:

- what you learned
- what you still do not trust
- where to resume next

If the answer is no, the session is not closed yet.
