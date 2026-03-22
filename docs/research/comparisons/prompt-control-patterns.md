# Prompt Control Patterns

## Comparison Goal

Compare how local-knowledge RAG systems use prompts to constrain retrieval, answer generation, and loop behavior.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- where prompts live
- dominant prompt role
- tool-use constraint style
- output-shaping strategy
- context injection style
- prompt complexity location

## Comparison Table

| Dimension | rag-skill | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- | --- |
| Main prompt location | `SKILL.md` plus file-type reference docs such as `pdf_reading.md` and `excel_analysis.md` | one backend prompt module with function-mode and ReAct system prompts | relatively light prompt surface; tool descriptions and response formatting carry much of the contract | large shared prompt library used across extraction, keywording, context formatting, and answer generation |
| Dominant prompt role | govern retrieval procedure, file routing, and pre-read gates before content work begins | tell the model how to navigate the knowledge map and when to call `retrieve_files` | describe tool semantics and format retrieval output for MCP clients | shape multiple internal LLM subfunctions: extraction, merging, keyword generation, and final answer synthesis |
| Tool-use constraint style | prompt-first operational rules like "read references first", "use `test -d`", and "max 5 rounds" | explicit rules in the system prompt plus one retrieval tool schema | tool contract is more protocol-first than prompt-first | mostly not tool-calling; prompt layer instead controls internal retrieval-related LLM tasks |
| Output-shaping strategy | mostly procedural, with little final-answer formatting beyond citing files and uncertainty | ReAct tags or native function-calling events; otherwise relatively light answer formatting | Markdown plus citation JSON blocks built mostly in code, not by elaborate prompting | strong prompt templates plus token-budgeted context templates and reference-section rules |
| Context injection style | inject directory/index knowledge by having the agent read `data_structure.md` files and only then open target files | inject one knowledge-map summary into the system prompt, then append raw retrieved observations | query string goes into retrieval core; final user-facing formatting is built from structured results | inject structured KG/chunk context into answer prompts and run separate prompts for keyword extraction and graph construction |
| Where prompt complexity lives | externalized into skill docs and operational references rather than code-built prompts | retrieval policy and loop format | very little in prompts; more in typed code, retrieval pipeline, and MCP protocol | heavily inside reusable prompt templates tied to many pipeline stages |

## Shared Patterns

- all four systems use prompts for retrieval-related control, not only for final natural-language answers
- none of the four relies mainly on few-shot prompting; structure, task decomposition, and code-side contracts do more of the work
- all four show that prompt design and retrieval design are tightly coupled in local-knowledge systems

## Key Differences

- `rag-skill` is the most documentation-governed system:
  prompt logic lives in a skill protocol and reference documents that teach the agent how to behave before any answer prompt is even assembled
- `deep-rag` is the most prompt-visible system:
  the model is directly instructed how to behave around the knowledge map and retrieval loop
- `MODULAR-RAG-MCP-SERVER` is the least prompt-dependent:
  most of its behavior is enforced by typed retrieval code, MCP tool schemas, and response builders rather than heavy LLM instruction stacks
- `LightRAG` is the most prompt-layered:
  prompts do not mostly control an external agent loop, but they do control many internal transformations that make the retrieval substrate work

## Conclusions

These four projects suggest four prompt-control patterns:

- documentation-governed workflow control
- prompt-guided retrieval navigation
- protocol-and-code-dominant retrieval control
- prompt-layered retrieval substrate shaping

That split matters because prompt complexity lands in different places.

- In `rag-skill`, prompt complexity is procedural and lives outside the runtime app surface.
- In `deep-rag`, prompt complexity is user-facing and loop-facing.
- In `MODULAR-RAG-MCP-SERVER`, prompt complexity is minimized and pushed into code and protocol contracts.
- In `LightRAG`, prompt complexity is internalized into the retrieval pipeline itself.

If the goal is to govern an agent's local-file workflow through docs and skill instructions, `rag-skill` is the clearest reference.

If the goal is to let the model actively steer retrieval inside one request, `deep-rag` is the clearest reference.

If the goal is to keep retrieval behavior stable by minimizing prompt responsibility, `MODULAR-RAG-MCP-SERVER` is the strongest reference.

If the goal is to use LLMs as internal operators inside a richer retrieval substrate, `LightRAG` is the most representative pattern.

## Open Questions

- How well does documentation-governed prompt control scale once `rag-skill` corpora and tool variance both grow?
- Can `deep-rag` keep its prompt-visible navigation contract while reducing ReAct-format fragility?
- Can `MODULAR-RAG-MCP-SERVER` stay prompt-light if it adds more retrieval or summarization behaviors?
- How much of `LightRAG`'s practical quality depends on prompt quality versus graph/vector substrate quality?
