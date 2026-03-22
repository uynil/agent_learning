# Prompt Function Mapping Patterns

## Comparison Goal

Compare how different agent projects map prompt functions to the concrete elements that make those functions work.

## Compared Projects

- [autoresearch](/Users/uynil/projects/agents/docs/research/projects/autoresearch/prompts-analysis.md)
- [DeepSearchAgent-Demo](/Users/uynil/projects/agents/docs/research/projects/DeepSearchAgent-Demo/prompts-analysis.md)
- [gpt-researcher](/Users/uynil/projects/agents/docs/research/projects/gpt-researcher/prompts-analysis.md)

## Candidate Dimensions

- target function
- key prompt element
- role framing
- instruction ordering
- examples or demonstrations
- schema or output contract
- tool-use rules
- reflection or self-check
- context injection

## Comparison Table

| Function | autoresearch | DeepSearchAgent-Demo | gpt-researcher |
| --- | --- | --- | --- |
| planning or decomposition | `program.md` decomposes the work into setup, baseline, logging, and infinite experiment loop steps | report-structure prompt constrains the report into up to five sections | task classification and query-generation prompts shape the research path, with outline generation available as a later option |
| search direction | not web-search oriented; prompt drives experiment direction by telling the agent how to choose and test code changes | dedicated search prompts with schema and reasoning fields | minimal JSON list prompt that requests three search queries |
| synthesis | the prompt synthesizes behavior into operational rules, while result selection happens through metric comparison and git state | first-summary and reflection-summary prompts revise one paragraph at a time | chunk summarization plus final report prompt over accumulated `research_summary` |
| reflection or critique | no dedicated critique prompt; keep-or-reset decision acts as a crude external judge | explicit reflection prompt asks what is missing and generates a new search query | little explicit reflection in the main path |
| formatting | summary and TSV examples constrain how results are parsed and logged | separate final formatting prompt plus manual fallback | report prompt directly aims at final Markdown output |
| dominant prompt element | operational shell instructions, tool-use rules, and persistence policy | stage-local schema contracts and narrow task framing | concise stage templates plus injected research summary and role persona |

## Shared Patterns

- all three systems use prompts for named workflow functions instead of one giant all-purpose prompt
- none of the three depends mainly on few-shot examples; they rely more on structure, ordering, and context injection
- each project shows a different place where prompts can carry system behavior:
  operational control, stage-local schema control, or light templated routing/synthesis

## Key Differences

- `autoresearch` uses prompts as an operations manual and external state machine more than as a content-generation scaffold
- `DeepSearchAgent-Demo` uses prompts to do more of the workflow control itself, especially around decomposition, search revision, and strict output shaping
- `gpt-researcher` uses prompts more sparingly and lets orchestration plus summarization carry more of the workload
- `DeepSearchAgent-Demo` has a clearer function-to-element link for reflection, `gpt-researcher` has a clearer function-to-element link for persona selection and final synthesis, and `autoresearch` has the clearest function-to-element link for autonomy and context throttling

## Conclusions

The current samples suggest three prompt-function styles:

- prompt-as-operations-manual
- prompt-heavy stage control
- orchestration-heavy execution with lighter prompt templates

This is a useful comparison axis because the three systems distribute responsibility between prompts, code, and tools very differently.

## Open Questions

- will `autoresearch` remain stable if its prompt-defined shell workflow gets more complex over time?
- Does prompt-heavy stage control remain stable under noisy live runs?
- Does lighter prompting in `gpt-researcher` make the system easier to maintain or just shift complexity elsewhere?
- As more projects are added, will these remain the two main poles or just two points on a spectrum?
