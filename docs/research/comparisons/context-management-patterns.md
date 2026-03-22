# Context Management Patterns

## Comparison Goal

Compare how different agent projects keep, retrieve, summarize, compress, and hand off context.

## Compared Projects

- [autoresearch](/Users/uynil/projects/agents/docs/research/projects/autoresearch/README.md)
- [DeepSearchAgent-Demo](/Users/uynil/projects/agents/docs/research/projects/DeepSearchAgent-Demo/context-management.md)
- [gpt-researcher](/Users/uynil/projects/agents/docs/research/projects/gpt-researcher/context-management.md)

## Candidate Dimensions

- memory layers
- retrieval strategy
- summary strategy
- context packing
- what gets dropped
- failure modes

## Comparison Table

| Dimension | autoresearch | DeepSearchAgent-Demo | gpt-researcher |
| --- | --- | --- | --- |
| Main working memory | current git commit, current `train.py`, and minimal parsed run metrics | paragraph-level `latest_summary` | global `research_summary` |
| Durable evidence layer | `results.tsv`, `run.log`, and branch history | `search_history` per paragraph plus full saved state JSON | saved `.txt` artifacts plus `visited_urls` and accumulated summaries |
| Retrieval style | explicit manual retrieval through shell commands like `grep` and `tail` | explicit manual scope reduction by current paragraph | search first, then scrape and summarize relevant pages |
| Context packing | redirect full output to log, then read only key metric lines or last failure trace | truncate results and pass only the current paragraph state into each step | chunk long pages, summarize chunks, then re-summarize into compact report context |
| Update rule | replace the candidate code state by keep-or-reset; append one row per run to the TSV | replace `latest_summary` after each reflection summary pass | append summarized source material into one growing report context |
| Forgetting or dropping | drops most live console output and keeps only compressed artifacts plus current-best code | drops broad history from active prompts and keeps only current paragraph context | drops raw page detail after chunk summarization and carries forward compressed summaries |
| Resumability shape | operational resume through repo state, but weak semantic resume | explicit state save/load, but manual resume path | file-based recovery hints, but not a first-class stateful resume flow |

## Shared Patterns

- all three projects avoid feeding raw full-context artifacts back into every next step
- all three compress working context into smaller handoff units before the next model or tool action
- all three show that context management is as much about what gets dropped as what gets preserved

## Key Differences

- `autoresearch` manages context operationally through files and shell filters, not mainly through in-memory research state
- `DeepSearchAgent-Demo` uses structured local state and explicit handoff objects
- `gpt-researcher` uses a looser accumulation model centered on one growing `research_summary`
- `DeepSearchAgent-Demo` optimizes for per-section continuity, while `gpt-researcher` optimizes for throughput across many sources, and `autoresearch` optimizes for keeping an autonomous coding loop from flooding its own context window

## Conclusions

These projects now suggest three context-management patterns:

- operational file-and-git context with shell-based filtering
- local structured memory with explicit handoff
- global summary accumulation with lightweight deduplication

That three-way split is likely more useful than the earlier two-way split because `autoresearch` shows a real pattern that does not fit the other two.

## Open Questions

- how robust is operational file-and-git context once `autoresearch` runs for many hours?
- Which pattern survives longer, noisier research tasks better in practice?
- Does the extra state structure in `DeepSearchAgent-Demo` meaningfully improve resumability?
- Does the lighter `gpt-researcher` summary stack flatten nuance too early on complex topics?
