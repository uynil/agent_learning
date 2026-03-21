# How to Analyze Context Management

## Goal

Understand how a project decides what information to keep, retrieve, summarize, pack into model context, and hand off across steps or rounds.

## Checklist

1. Find state, memory, retrieval, transcript, cache, and scratchpad modules
2. Distinguish transient context from persistent memory
3. Map what enters each model call
4. Check how relevant context is selected, ranked, or filtered
5. Check whether summaries or compressed memories are created
6. Check how context is handed across agents, stages, or rounds
7. Check what is dropped, evicted, or forgotten
8. Note failure modes from stale, missing, or overloaded context

## Questions to Ask

- What information survives one step, one round, or one session?
- Is memory global, per-task, per-agent, or per-user?
- How is relevant context chosen?
- Who writes summaries or memory updates?
- How is context bounded for long tasks?
- What breaks first when context grows too large?

## Output Pattern

- Context Sources
- Memory and State Layers
- Retrieval Strategy
- Context Packing Strategy
- Update Rules
- Failure Modes
- Reusable Patterns
- Open Questions
