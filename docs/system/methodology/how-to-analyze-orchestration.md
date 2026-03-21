# How to Analyze Orchestration

## Goal

Understand how a project coordinates model calls, tools, intermediate state, and retries.

## Checklist

1. list all model call sites
2. map the order of calls
3. identify role splits
4. identify iteration or reflection loops
5. identify structure enforcement
6. identify retry or fallback paths
7. identify where the orchestration leaks into prompts

## Core Questions

- Which stage plans the work?
- Which stage executes the work?
- Which stage judges or revises the work?
- What happens when outputs are malformed?
- What happens when search or tool data is missing?
- Is the orchestration easy to extend?

## Output Pattern

- LLM Call Map
- Scheduling Pattern
- Reliability Mechanisms
- Tradeoffs
- Failure Modes
- Open Questions

