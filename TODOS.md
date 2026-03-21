# TODOs

Current pending TODOs: none.

The initial follow-up items from the first plan review were completed and are kept below for traceability.

## 1. Add a Session-Close Checklist

**Status:** done on 2026-03-21

**What:** Add a reusable checklist for ending a research session.

**Why:** The current workflow defines the broad stages well, but there is still no single close-out artifact that answers "what must be true before this research round counts as done?"

**Pros:**
- reduces missed updates across project notes, iteration logs, and system docs
- turns review decisions into an actionable end-of-session routine
- creates a stable base for future automation

**Cons:**
- introduces one more maintained document
- will need occasional updates when the workflow changes

**Context:** This should support the new rule split established in the review:
- high-frequency updates happen every session
- low-frequency updates happen only when promotion thresholds are met

It should likely live alongside the workflow docs instead of becoming a competing process definition.

**Depends on / blocked by:** Depends on the workflow source-of-truth decision staying in `docs/00-system/learning-workflow.md`. Not blocked by code or project research.

## 2. Add Prompt Smoke Cases

**Status:** done on 2026-03-21

**What:** Create a small set of fixed prompt regression cases for manual prompt validation.

**Why:** The repository has a prompt library and an evaluation rubric, but no stable benchmark scenarios. Without fixed cases, prompt changes are judged mostly by intuition.

**Pros:**
- makes prompt changes comparable across iterations
- reduces false confidence from one-off successful runs
- works well with the existing evaluation rubric

**Cons:**
- requires choosing representative research scenarios
- the case set may need occasional refresh as the system evolves

**Context:** Start with manual smoke cases rather than automation. A small set of 4-6 research scenarios is enough to catch prompt drift early without adding engineering overhead.

**Depends on / blocked by:** Best paired with the session-close checklist so smoke testing can be part of the regular close-out routine. Not otherwise blocked.

## 3. Add a Lightweight Topic Index

**Status:** done on 2026-03-21

**What:** Add a topic-oriented index page that links themes to project notes and comparison docs.

**Why:** The current structure is good for storage by document type, but weaker for retrieval by research topic. As more projects are added, finding "which projects demonstrate pattern X?" will get slower.

**Pros:**
- improves long-term discoverability without restructuring the repo
- helps cross-project comparison start from evidence instead of memory
- can evolve into a research map over time

**Cons:**
- adds an extra navigation document to maintain
- will decay if it is not updated when notes grow

**Context:** This should be a thin navigation layer, not a replacement for `docs/02-comparisons/`. A good initial location would be `docs/00-system/research-index.md` or `docs/02-comparisons/index.md`.

**Depends on / blocked by:** Depends on project notes gradually becoming richer. Not blocked by any code changes.
