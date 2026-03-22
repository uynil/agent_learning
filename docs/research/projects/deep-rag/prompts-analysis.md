---
project: deep-rag
source_path: examples/deep-rag
status: in-progress
confidence: medium
last_updated: 2026-03-22
next_action: compare executed prompt sources against prompt reference artifacts and frontend parsing assumptions, without adding more live runs
---

# Prompts Analysis

## Prompt Inventory

- `_create_base_system_prompt()`
  Shared system prompt that injects the knowledge-map summary and retrieval rules.
- `create_system_prompt()`
  Function-calling mode system prompt.
- `create_react_system_prompt()`
  ReAct-mode system prompt with format instructions and examples.
- summary-generation prompt in `Knowledge-Base-File-Summary/generate.py`
  Preprocessing prompt for file summaries and chunk plans.
- `Knowledge-Base-File-Summary/Summary Prompt.md`
  Reference prompt document that explains the summarization approach, but is not the prompt source actually executed by the generator.

## Prompt Function Map

| Prompt | Primary Function | Key Output |
| --- | --- | --- |
| base runtime system prompt | define answer policy and retrieval policy | tool-aware system instruction |
| ReAct system prompt | teach thought/action format for non-native tool calling | formatted action loop |
| summary-generation prompt | compress files into a knowledge map | short file summaries or chunk plans |
| large-file summary-generation prompt | split oversized files semantically and summarize each chunk | JSON chunk list with summaries |

## Prompt Structure

- The runtime prompt is one large system prompt plus conversation history.
- The core structure is:
  - answer policy
  - tool policy
  - current time
  - full knowledge-base file summary
  - retrieval tool contract
  - examples
- ReAct mode adds an explicit output grammar on top of the same base prompt.
- Preprocessing prompts are user-only task prompts aimed at summarization, not runtime conversation.

## Core Prompt Elements

- Role stance: answer only from the knowledge base.
- Tool-use boundary: retrieve files when the summary is insufficient.
- Refusal rule: answer "I don't know" only after diligent retrieval.
- Knowledge-map injection: the full `summary.txt` is inserted directly into the system prompt.
- Retrieval schema: `{"file_paths": ["path1", "path2"]}`.
- Examples: file, directory, mixed, and `/` retrieval examples.
- ReAct grammar: `<|Thought|>`, `<|Action|>`, `<|Action Input|>`, `<|Observation|>`, `<|Final Answer|>`.
- Preprocessing compression rule: keep summaries extremely short and information-dense.
- Small-file summary budget is capped at fewer than 10 tokens.
- Large-file chunk summaries also target fewer than 10 tokens per chunk descriptor.

## Function-to-Mechanism Mapping

| Function | Prompt Element | Why It Works |
| --- | --- | --- |
| global knowledge orientation | file-summary injection | it gives the model a navigable map before any retrieval happens |
| constrain answer source | "answers must strictly come from the knowledge base" | it narrows the trust boundary |
| trigger active retrieval | retrieval rules + file path examples | it turns file selection into an explicit model decision |
| support non-native tool calling | ReAct output grammar and examples | it gives weaker models a prompt-only tool protocol |
| enable semantic chunking during preprocessing | line-numbered large-file prompt with token bounds | it asks the model to split large files by meaning rather than raw size only |
| reduce low-signal map entries | summary prompt bans low-information words and file-path words | it pushes summaries toward discriminative content |

## Prompt Organization Pattern

- runtime prompts are centralized in one backend prompt module
- preprocessing prompts live inside the summary generator
- `Summary Prompt.md` is adjacent documentation, not the runtime or preprocessing source of truth
- runtime prompt organization is mode-based:
  - shared base prompt
  - function-calling mode
  - ReAct mode
- tool protocol and knowledge-map policy are more important than role personas

## Prompt Composition Strategy

- runtime prompt = static instruction block + current time + generated file summary + tool examples
- ReAct prompt = base runtime prompt + explicit reasoning/action format
- preprocessing prompt = generated task instructions + file content + token-budget constraints

## Prompt-to-Code Coupling

- `backend/main.py` chooses between `create_system_prompt()` and `create_react_system_prompt()`.
- `backend/prompts.py` owns both the prompt text and the tool schema.
- `backend/react_handler.py` depends on exact ReAct tag patterns that `prompts.py` teaches.
- `Knowledge-Base-File-Summary/generate.py` embeds prompt logic directly inside the preprocessing pipeline.
- The frontend stream parser in `App.tsx` also depends on the ReAct prompt contract, because it looks for `<|Final Answer|>` before rendering the assistant's final text.

## Constraint and Reliability Strategy

- strict knowledge-boundary instruction
- explicit retrieval tool schema
- example-driven path selection
- bounded iteration counts in runtime loops
- dense knowledge-map summaries to improve retrieval choices
- ReAct examples to stabilize text-only tool calling
- frontend-side `<|Final Answer|>` extraction to avoid showing raw ReAct scaffolding in the visible chat transcript

## Likely Failure Modes

- the model may over-trust the file summary and skip retrieval too early
- retrieving `/` or large directories may flood the context window
- ReAct format may drift and fail parsing
- the summary generator may produce over-compressed file summaries that hide distinctions
- documented env keys for knowledge-base paths may not line up with actual settings field names
- prompt reference docs may drift from executed prompt strings because preprocessing prompt text is hardcoded in `generate.py`
- summarization runs may expose source corpus contents in logs because the generator prints full prompts before calling the model

## Reusable Prompt Patterns

- inject a compact "knowledge map" before active retrieval
- use file-path examples to teach tool-selection behavior
- separate runtime prompting from preprocessing summarization prompting
- support both native function calling and prompt-only ReAct with shared base instructions
- use ultra-short summaries as retrieval hints rather than as final answers
- distinguish between executable prompt sources and prompt reference artifacts when documenting a project
