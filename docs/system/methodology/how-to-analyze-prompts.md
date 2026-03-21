# How to Analyze Prompts

## Goal

Understand prompts as part of a system, not as isolated text snippets.

Prompt analysis should cover more than reliability. It should explain function, structure, core elements, composition, coupling, mechanisms, and failure modes.

## Checklist

1. Find where prompts are stored
2. Identify prompt layers:
   - system
   - developer
   - user
   - tool instructions
   - templates or generated fragments
3. Group prompts by purpose
4. For each important prompt, identify its intended function:
   - planning
   - routing
   - tool selection
   - synthesis
   - critique
   - formatting
   - reflection
5. Inspect prompt structure and core elements:
   - role
   - task
   - context
   - constraints
   - examples
   - output contract
   - tool-use policy
   - reflection hooks
6. Map which mechanisms likely implement each function:
   - role framing
   - instruction ordering
   - examples
   - schemas
   - tool rules
   - decomposition
   - reflection
   - context injection
7. Check whether prompts are static or dynamically assembled
8. Check how prompts connect to code
9. Note output constraints and guardrails
10. Note tool-use instructions
11. Note reflection, retry, or repair patterns
12. Infer the author's design intent

## Questions to Ask

- Are prompts organized by role, stage, or feature?
- What is each important prompt trying to make the model do?
- Which mechanism is carrying most of that effect?
- Which instructions are critical and which are ornamental?
- What are the stable core elements across prompts?
- Which parts are hard-coded and which parts are injected dynamically?
- Is the prompt static or generated dynamically?
- Is the prompt responsible for structure, safety, or both?
- Is there a consistent prompt skeleton across the system?
- How brittle is the output contract?
- What would likely break first?

## Output Pattern

- Prompt Inventory
- Prompt Function Map
- Prompt Structure
- Core Prompt Elements
- Function-to-Mechanism Mapping
- Organization Pattern
- Composition Strategy
- Coupling with Code
- Constraint and Reliability Strategy
- Failure Modes
- Reusable Patterns
