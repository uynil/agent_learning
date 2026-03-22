# Configuration Surface Patterns

## Comparison Goal

Compare how local-knowledge RAG systems expose provider, storage, path, and runtime configuration, and how risky those configuration surfaces are in practice.

## Compared Projects

- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- primary config medium
- provider surface
- storage surface
- live reload or propagation model
- docs/runtime alignment
- dominant config risk

## Comparison Table

| Dimension | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Primary config medium | `.env` plus backend settings object and in-app config editor | `config/settings.yaml` plus Python settings dataclasses | CLI args, env vars, workspace settings, and backend-specific config combinations |
| Provider surface | several provider names exposed, but backend model override wiring is partial | simpler checked-in provider defaults, mainly around embedding and LLM config | broadest provider surface: multiple LLM, embedding, rerank, and storage bindings |
| Storage/path surface | knowledge-base path, chunk path, summary path, and frontend API base URL assumptions | vector-store provider, persist directory, collection, BM25 index paths, and trace paths | many storage backends, workspace isolation, multiple stores, and shared namespace state |
| Live reload model | partial and uneven; some env changes refresh, some settings-dependent singletons likely stay stale | mostly restart-oriented and file-backed; clearer than `deep-rag`, but less dynamic | broad runtime configurability, but effective health depends on many backend/storage/provider combinations |
| Docs/runtime alignment | weakest of the three; documented env names and some frontend claims drift from actual runtime behavior | medium; defaults and tests/config assumptions drift, but the core settings model is explicit | medium; surface is broad and explicit, but complexity makes practical behavior depend on chosen combination |
| Strongest config strength | simple default mental model for a small app | explicit typed settings and repo-anchored path resolution | most flexible and scalable config surface |
| Dominant config risk | split runtime state and misleading hot-reload semantics | bootstrap/config drift between defaults, dependencies, and expected runtime path | combinatorial complexity across providers, storages, modes, and deployment surfaces |

## Reload and Propagation

| Question | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Can config plausibly change during runtime? | partly, but propagation is uneven across modules | usually treated as startup configuration | yes in many dimensions, but practical correctness depends on the specific stack chosen |
| What is the main propagation problem? | updating one `settings` handle does not guarantee imported singletons or module-level state refresh | users may think the packaged entrypoint represents the real runtime, but launcher/config/bootstrap drift says otherwise | many independent knobs make it harder to know which configuration combination is actually healthy |
| What does the user most need to trust? | that "save and apply" really changes the live runtime | that documented defaults, dependencies, and entrypoints line up | that a chosen provider/storage/backend combination is actually supported and healthy together |

## Shared Patterns

- all three projects expose more configuration than a narrow library would
- all three show that configuration quality affects system trust as much as retrieval quality does
- all three separate "what the code can theoretically support" from "what the default runtime reliably uses"

## Key Differences

- `deep-rag` has the smallest configuration surface, but also the leakiest hot-reload story and the clearest docs/runtime naming drift
- `MODULAR-RAG-MCP-SERVER` has a more explicit typed settings model, but the real risk is that bootstrap, defaults, and launcher path do not fully line up
- `LightRAG` has the broadest and most powerful configuration surface, but that power comes with the highest combination complexity

## Conclusions

These projects suggest three configuration-surface patterns:

- small but leaky app config surface
- explicit typed service config surface
- broad platform config matrix

That split matters because each one fails differently.

- In small but leaky app config surfaces, the main risk is false confidence that runtime changes have propagated when they have not.
- In explicit typed service config surfaces, the main risk is bootstrap drift around otherwise clear configuration objects.
- In broad platform config matrices, the main risk is not missing knobs but too many valid-looking combinations whose runtime health is hard to reason about.

If the goal is a small local app, `deep-rag` shows the simplest starting point, but also a good warning about config propagation and UI promises.

If the goal is a service with a relatively legible settings core, `MODULAR-RAG-MCP-SERVER` is the strongest reference.

If the goal is a platform intended to support many deployment shapes and backends, `LightRAG` is the strongest reference.

## Open Questions

- Can `deep-rag` centralize runtime settings enough that its config editor becomes trustworthy?
- Can `MODULAR-RAG-MCP-SERVER` align launcher, dependencies, defaults, and tests into one coherent bootstrap path?
- Can `LightRAG` make the healthy subset of its huge config matrix more obvious to operators?
