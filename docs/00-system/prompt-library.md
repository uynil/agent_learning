# Prompt Library

These prompts are written for ongoing study of open-source agent projects. They are designed to work across general-purpose LLM tools and coding assistants.

## Shared Output Rules

All research prompts should follow these rules:

- prioritize code and docs over guesses
- cite concrete files, modules, classes, or functions when possible
- separate `Observed Facts`, `Inferences`, `Open Questions`, and `Improvement Ideas`
- prefer structured output over long unstructured essays
- say when the evidence is incomplete

## 1. Master Prompt

```text
You are an open-source agent project research assistant.

Your task is to help me study one or more agent projects with a long-term learning goal.
Focus on:
1. architecture design
2. LLM orchestration and scheduling
3. prompt design, organization, and usage
4. reusable lessons and open questions

Rules:
- prioritize evidence from repository files over generic explanations
- separate:
  - Observed Facts
  - Inferences
  - Open Questions
  - Improvement Ideas
- do not pretend uncertainty is solved
- optimize for reusable notes, not one-off chat output

At the beginning, classify the task into one mode:
- Quick Scan
- Deep Dive
- Cross Project Compare
- Prompt Audit
- Doc Refine

Then output:

# Research Plan Card
- Mode:
- Goal:
- Scope:
- Key Files to Read:
- Expected Outputs:
- Risks / Unknowns:
- Recommended Next Step:
```

## 2. Quick Scan Prompt

```text
Please do a Quick Scan of this agent project.

Goals:
1. summarize what problem the project solves
2. identify the core entry points, core agents, core prompts, core state, and core tools
3. describe the main execution flow in plain language
4. classify the project style:
   - single agent
   - planner / executor
   - graph / workflow
   - multi-agent collaboration
   - tool-using agent
   - hybrid
5. identify the three most valuable files for a deeper read
6. recommend whether to stop here or continue into a deep dive

Output sections:
- Project Summary
- Key Files
- Execution Flow
- Agent Pattern
- Observed Facts
- Inferences
- Open Questions
- Next Step
```

## 3. Architecture Prompt

```text
Please analyze the architecture design of this agent project.

Focus on:
1. main modules and their responsibilities
2. module boundaries
3. execution flow
4. state and data flow
5. extension points
6. likely strengths and tradeoffs

Output sections:
- Architecture Summary
- Module Breakdown
- Execution Flow
- State and Data Flow
- Extensibility
- Tradeoffs
- Open Questions
```

## 4. LLM Orchestration Prompt

```text
Please analyze the LLM orchestration design of this project.

Focus on:
1. where the model is called
2. what each call is responsible for
3. whether there are planner, executor, reviewer, or reflector roles
4. whether calls are serial, parallel, or iterative
5. whether output is constrained with schema, parser, or validation
6. whether there are retry, fallback, or recovery mechanisms
7. how tightly prompts are coupled to orchestration code
8. what task types this orchestration is well suited for

Output sections:
- LLM Call Map
- Scheduling Pattern
- Structured Output Strategy
- Reliability Mechanisms
- Strengths
- Limitations
- Questions for Further Research
```

## 5. Prompt Audit Prompt

```text
Please analyze the prompt design and usage patterns in this project.

Focus on:
1. where prompts are stored
2. whether prompts are organized by role or by workflow stage
3. how prompts are connected to code
4. key constraints in the prompts:
   - role
   - task
   - output shape
   - boundaries
   - tool-use constraints
   - reflection mechanisms
5. likely failure modes
6. reusable prompt patterns worth borrowing

Output sections:
- Prompt Inventory
- Prompt Organization Pattern
- Prompt-to-Code Coupling
- Key Prompt Techniques
- Likely Failure Modes
- Reusable Prompt Patterns
- My Learning Notes
```

## 6. Doc Refine Prompt

```text
Please turn the current research output into stable study notes that can be updated later.

Required sections:
- Project Overview
- Confirmed Facts
- My Inferences
- Architecture Flow
- LLM Orchestration Analysis
- Prompt Analysis
- Key Files
- Open Questions
- Next Research Plan
- Personal Takeaways

Writing style:
- concise
- structured
- easy to extend
- explicit about uncertainty
```

## Suggested Usage

- Start with `Master Prompt`
- Run `Quick Scan` on a new project
- Use `Architecture Prompt` and `LLM Orchestration Prompt` for deeper study
- Use `Prompt Audit Prompt` when the project has interesting prompt design
- End with `Doc Refine Prompt`
- Re-check key prompts against [prompt-smoke-cases.md](./prompt-smoke-cases.md) after meaningful prompt edits
