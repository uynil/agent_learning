# Prompt Library

These prompts are written for ongoing study of open-source agent projects. They are designed to work across general-purpose LLM tools and coding assistants.

## Shared Output Rules

All research prompts should follow these rules:

- prioritize code and docs over guesses
- cite concrete files, modules, classes, or functions when possible
- separate `Observed Facts`, `Inferences`, `Open Questions`, and `Improvement Ideas`
- prefer structured output over long unstructured essays
- say when the evidence is incomplete
- when analyzing prompts, inspect function, structure, core elements, composition, function-to-mechanism mapping, and failure modes

## 1. Master Prompt

```text
You are an open-source agent project research assistant.

Your task is to help me study one or more agent projects with a long-term learning goal.
Focus on:
1. architecture design
2. LLM orchestration and scheduling
3. prompt structure, core elements, function-to-mechanism mapping, organization, and usage
4. context management, memory, and retrieval strategy
5. deep research loops such as question rewrite, decomposition, multi-round synthesis, and stop conditions
6. reusable lessons and open questions

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
- Prompt Audit
- Context Management Audit
- Deep Research Mode
- Cross Project Compare
- Doc Refine

Then output:

# Research Plan Card
- Mode:
- Primary Theme:
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
7. call out whether context management or deep research loops look important enough for a later pass

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
8. how context is passed between calls or rounds
9. what task types this orchestration is well suited for

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
2. how prompts are layered or composed:
   - system
   - developer
   - user
   - tool instructions
   - templates or builders
3. what each prompt is trying to make the model do:
   - classify
   - plan
   - select tools
   - synthesize
   - critique
   - format output
   - reflect
4. prompt structure and core elements:
   - role
   - task
   - context
   - constraints
   - examples
   - output shape or schema
   - tool-use policy
   - reflection or self-check
5. how those functions are implemented:
   - role framing
   - instruction ordering
   - examples
   - schemas or format contracts
   - tool rules
   - decomposition steps
   - reflection steps
   - context injection
6. whether prompts are organized by role, stage, feature, or lifecycle
7. whether prompts are static, parameterized, or dynamically assembled
8. how prompts are connected to code
9. constraint, guardrail, and reliability strategy
10. likely failure modes
11. reusable prompt patterns worth borrowing

Output sections:
- Prompt Inventory
- Prompt Function Map
- Prompt Structure
- Core Prompt Elements
- Function-to-Mechanism Mapping
- Prompt Organization Pattern
- Prompt Composition Strategy
- Prompt-to-Code Coupling
- Constraint and Reliability Strategy
- Likely Failure Modes
- Reusable Prompt Patterns
- My Learning Notes
```

## 6. Context Management Prompt

```text
Please analyze the context management design of this project.

Focus on:
1. where context comes from:
   - user input
   - state objects
   - memory stores
   - retrieval systems
   - summaries
   - prior model outputs
2. what context is transient versus persistent
3. how relevant context is selected, ranked, or filtered
4. how context is packed for model calls
5. how context is updated after each step or round
6. whether there are summaries, caches, scratchpads, or memory layers
7. what gets dropped, compressed, or forgotten
8. likely failure modes from context overload, stale context, or missing context

Output sections:
- Context Sources
- Memory and State Layers
- Retrieval and Selection Strategy
- Context Packing Strategy
- Context Update Rules
- Failure Modes
- Reusable Patterns
- Open Questions
```

## 7. Deep Research Prompt

```text
Please analyze how this project supports deep research or multi-round investigation.

Focus on:
1. how a broad user question is interpreted
2. whether the question is rewritten, decomposed, or split into subquestions
3. how the project loops through search, reading, synthesis, and revision
4. how evidence is accumulated and tracked across rounds
5. whether there are reviewer, critic, or reflection stages
6. how each round is summarized for the next round
7. what the stop conditions are
8. whether the process is resumable after interruption

Output sections:
- Research Loop Overview
- Question Rewrite Strategy
- Decomposition Strategy
- Evidence Accumulation
- Reflection and Revision
- Stop Conditions
- Resumability and Context Handoff
- Reusable Patterns
```

## 8. Question Rewrite Prompt

```text
Please rewrite this broad agent-research question into a sharper set of research questions.

Goals:
1. identify the real research target
2. split the question into the smallest useful subquestions
3. point each subquestion to the best next files or code areas
4. define what evidence would count as an answer
5. define when this round should stop

Output sections:
- Original Question
- Rewritten Core Question
- Subquestions
- Evidence Needed
- Best Next Files
- Stop Condition
```

## 9. Round Summary Prompt

```text
Please summarize the current research round so it can be resumed later without rereading the whole session.

Output sections:
- Current Core Question
- Rewritten Question
- Files Checked
- Confirmed Facts
- Working Hypothesis
- Disconfirmed Ideas
- Remaining Gaps
- Next Round Question
- Resumable Context Summary
```

## 10. Doc Refine Prompt

```text
Please turn the current research output into stable study notes that can be updated later.

Required sections:
- Project Overview
- Confirmed Facts
- My Inferences
- Architecture Flow
- LLM Orchestration Analysis
- Prompt Structure and Analysis
- Context Management Analysis
- Deep Research Loop Notes
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
- Use `Prompt Audit Prompt` when the project has interesting prompt structure or prompt design
- Use `Context Management Prompt` when memory, retrieval, summaries, or long-context handling matter
- Use `Deep Research Prompt` when the project appears to do query rewrite or multi-round investigation
- Use `Question Rewrite Prompt` at the start of a broad or fuzzy deep-research session
- Use `Round Summary Prompt` at the end of each deep-research round
- End with `Doc Refine Prompt`
- Re-check key prompts against [prompt-smoke-cases.md](./prompt-smoke-cases.md) after meaningful prompt edits
