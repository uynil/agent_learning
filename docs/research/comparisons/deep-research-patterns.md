# Deep Research Patterns

## Comparison Goal

Compare how different agent projects support broad-question investigation through query rewrite, decomposition, multi-round synthesis, and stop conditions.

## Compared Projects

- [autoresearch](/Users/uynil/projects/agents/docs/research/projects/autoresearch/README.md)
- [DeepSearchAgent-Demo](/Users/uynil/projects/agents/docs/research/projects/DeepSearchAgent-Demo/deep-research-mode.md)
- [gpt-researcher](/Users/uynil/projects/agents/docs/research/projects/gpt-researcher/README.md)

## Candidate Dimensions

- query rewrite
- subquestion generation
- evidence accumulation
- reflection or critique
- resumability
- stop conditions

## Comparison Table

| Dimension | autoresearch | DeepSearchAgent-Demo | gpt-researcher |
| --- | --- | --- | --- |
| Broad-question handling | does not target broad knowledge questions; targets repeated model-improvement experiments under fixed constraints | turns the query into report sections | turns the query into search queries and then gathers broad evidence |
| Query rewrite | none in the knowledge-research sense; new experiment ideas are generated implicitly by the agent | local rewrite happens through reflection on the current paragraph state | query generation exists, but not as an iterative rewrite loop |
| Decomposition style | no explicit question tree; decomposition is over successive code experiments | section-oriented decomposition with up to five paragraphs | breadth-first search decomposition rather than question-tree decomposition |
| Evidence accumulation | `results.tsv`, `run.log`, and git history accumulate experiment evidence | per-paragraph `search_history` plus evolving `latest_summary` | global `research_summary` built from summarized sources |
| Reflection or critique | no first-class reflection stage beyond the agent's own judgment of keep vs reset | explicit reflection stage per paragraph | no strong reflective revision stage in the main path |
| Stop conditions | indefinite until interrupted, with per-run success/failure judgment | bounded by `max_reflections` and paragraph completion | stops after source gathering and final synthesis, without explicit revision rounds |
| Resumability | operationally resumable through branch state and result logs, but weak semantic memory | structured saved state, but manual resume flow | file-based artifacts, weaker first-class resume model |

## Shared Patterns

- all three projects avoid answering directly from the initial input and instead build intermediate evidence artifacts first
- all three show that “research” can mean different loop shapes:
  experiment iteration, sectioned reflection, or source aggregation

## Key Differences

- `autoresearch` is only “research” in the experimental optimization sense; it is not a broad-question investigation system
- `DeepSearchAgent-Demo` looks like a true multi-round deep-research workflow, but the rounds are local to each section
- `gpt-researcher` looks more like a breadth-first research aggregator than a first-class deep-research loop
- `DeepSearchAgent-Demo` makes revision explicit; `gpt-researcher` makes collection breadth explicit; `autoresearch` makes selection pressure explicit

## Conclusions

With current evidence, `DeepSearchAgent-Demo` remains the best reference sample for “deep research mode” in this repository.

`gpt-researcher` is valuable as a contrast case:
it shows a strong research pipeline without a rich iterative reflection loop.

`autoresearch` is valuable as a second contrast case:
it shows that autonomous research can also mean experimental search over code changes, which should not be conflated with deep question decomposition.

## Open Questions

- should the repository explicitly distinguish “experimental autoresearch” from “deep research” as separate comparison families?
- After a live run, does `DeepSearchAgent-Demo` still deserve the “deep research” label?
- Could `gpt-researcher` gain a stronger deep-research identity with only a reviewer or rewrite stage added?
- Will future projects in this repo show question-tree decomposition rather than section-based decomposition?
