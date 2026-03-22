# State Management Patterns

## Goal

Compare how projects represent state, pass context, and persist intermediate results.

## Comparison Dimensions

- state objects or schemas
- mutable vs immutable flow
- persistence boundaries
- context handoff
- failure recovery impact

## Comparison Table

| Project | State Shape | Persistence | Context Handoff | Recovery | Notes |
| --- | --- | --- | --- | --- | --- |
| BettaFish | TBD | TBD | TBD | TBD | not started |
| DeepSearchAgent-Demo | explicit nested state tree: query, paragraphs, per-paragraph research state | saved JSON snapshots plus in-memory structured objects | `latest_summary` and `search_history` handed from node to node at paragraph scope | manual save/load is available and materially useful | strongest explicit state model among the currently compared research agents |
| deep-rag | mostly stateless backend flow plus frontend-replayed messages, `summary.txt`, file-system artifacts, and a partially split runtime settings state after in-app config edits | knowledge-base files, summary artifacts, mirrored markdown chunks, `.env`, and client-side message history | full prior messages are resent by the frontend; tool observations are appended back into the loop; some runtime config is refreshed through `backend.main` while other modules still hold older settings references | operationally easy to resume if artifacts exist, but weak server-side session recovery and uneven live-config propagation | good example of file-backed retrieval with thin server-side memory and a leaky hot-reload story |
| LightRAG | single control-plane object over many stores plus workspace-scoped shared-storage substrate | graph store, KV stores, vector stores, cache stores, document-status store, and shared namespace state | query modes and structured retrieval outputs hand off entities, relations, chunks, and references | strong durable state, but recovery semantics depend on many backend/storage components staying consistent | richest runtime state model among the current RAG systems |
| MiroFish | TBD | TBD | TBD | TBD | not started |
| MODULAR-RAG-MCP-SERVER | typed contracts plus mostly file-backed operational state around collections and traces | repo-local config, BM25 indexes, vector-store collections, and JSONL traces | per-query retrieval results and traces are produced by rebuilt collection-scoped components | good artifact-level recovery, but bootstrap and launcher drift make "resume by rerun" less clean than the core architecture suggests | retrieval core is modular, but state is still mostly externalized into files and stores |
| autoresearch | operational state split across git branch, current commit, `train.py`, `results.tsv`, and `run.log` | repo state plus untracked local artifacts | shell commands retrieve only needed metrics or traces from files | easy operational resume, weak semantic memory | unusual example where state is mostly externalized into tools and files rather than an object model |
| chatwoot | TBD | TBD | TBD | TBD | not started |
| everything-claude-code | TBD | TBD | TBD | TBD | not started |
| gpt-researcher | lightweight in-process state around `research_summary` and `visited_urls` | output files under `./outputs/<hash>` plus accumulated summaries | summarized page content appended into one growing report context | some file-based continuation hints, but weak formal resume model | favors lightweight accumulation over explicit state schemas |

## Notes

The current evidence suggests three distinct state-management styles worth tracking:

- explicit structured state
- lightweight accumulated summary state
- operational file-and-git state

Recent RAG comparisons suggest at least three more retrieval-oriented variants:

- frontend-replayed file-backed runtime state
- file-backed modular service state
- shared multi-store runtime state

Recent static analysis of `deep-rag` adds one more wrinkle worth tracking:

- partially split runtime-config state, where `.env` edits may update one settings handle but not every singleton or imported settings reference
