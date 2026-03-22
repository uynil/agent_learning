# Serving Surface Patterns

## Comparison Goal

Compare how local-knowledge RAG systems are exposed to users and other programs, and how much architectural complexity lives above the retrieval core.

## Compared Projects

- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)
- [MODULAR-RAG-MCP-SERVER](/Users/uynil/projects/agents/docs/research/projects/MODULAR-RAG-MCP-SERVER/README.md)
- [LightRAG](/Users/uynil/projects/agents/docs/research/projects/LightRAG/README.md)

## Comparison Dimensions

- primary interface
- auxiliary interface surfaces
- runtime contract shape
- frontend or client coupling
- deployment assumptions
- strongest serving risk

## Comparison Table

| Dimension | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Primary interface | FastAPI backend with chat SSE endpoints | stdio MCP server | FastAPI server over core library |
| Auxiliary surfaces | React frontend and config editor | placeholder root CLI entrypoint plus repo docs/spec shell | WebUI, Ollama-compatible API, auth, and broad CLI/env configuration |
| Runtime contract shape | HTTP endpoints plus SSE stream semantics for function and ReAct modes | MCP `initialize`, `tools/list`, and `tools/call` over stdio | HTTP routes exposing query modes, document operations, graph operations, and server config |
| Client coupling shape | high: frontend buffers ReAct protocol markers, replays full chat history, and compensates for backend protocol details in the UI | medium: MCP protocol is clean, but package entrypoint still points to placeholder launcher | medium to high: server is broad and configurable, but clients must understand many modes, bindings, and storage options |
| Deployment assumption shape | backend plus frontend are expected together, some frontend paths still assume localhost, and in-app config save/apply is only partially centralized | packaged launcher suggests one path while the real MCP runtime lives deeper in `src/` | designed for many deployment shapes, including local server, WebUI, and alternate API compatibility layers |
| Strongest serving strength | visible tool flow and straightforward backend/frontend story once the app is running | clean protocol surface when using the real MCP module entrypoint | richest serving surface and most reusable platform boundary of the three |
| Strongest serving risk | frontend/backend protocol coupling plus partial config-propagation semantics after in-app `.env` edits | launcher drift and dependency drift hide the real serving surface | serving complexity expands quickly because API, WebUI, storage matrix, and provider matrix all interact |

## Surface-to-Core Relationship

| Question | deep-rag | MODULAR-RAG-MCP-SERVER | LightRAG |
| --- | --- | --- | --- |
| Does the default entrypoint reach the real core? | mostly yes for the backend app | no; packaged root path still lands on placeholder launcher | yes, but there are many optional surfaces above the same core |
| Is the serving layer thin? | no; the frontend meaningfully shapes the user-visible contract | mixed; MCP layer is reasonably thin, but bootstrap packaging is misleading | no; the serving layer is a substantial product surface in its own right |
| Where is the main contract fragility? | SSE marker conventions, frontend assumptions, and config updates that do not clearly propagate across all backend modules | package entrypoint, dependency ladder, and tool-edge abstraction leaks | configuration combinatorics and divergence across bindings or storage backends |

## Shared Patterns

- all three projects have more serving complexity than a pure retrieval library
- all three expose retrieval through a user-facing or program-facing runtime contract rather than only through internal Python calls
- all three show that serving shape can drift from core retrieval quality: a good retrieval core does not automatically imply a clean default user experience

## Key Differences

- `deep-rag` is the most visibly app-shaped system:
  its frontend, SSE semantics, and backend loop all matter directly to the product experience
- `deep-rag` also has the most obvious "UI says one thing, runtime may do another" risk:
  the config editor promises immediate application, but the code path refreshes settings unevenly across modules
- `MODULAR-RAG-MCP-SERVER` is the most protocol-centered:
  its best serving surface is MCP over stdio, but the packaged launcher still obscures that fact
- `LightRAG` is the broadest platform surface:
  it is not only a retrieval engine, but also a server, WebUI host, and compatibility layer for multiple deployment and provider shapes

## Conclusions

These three projects suggest three serving-surface patterns:

- app-coupled retrieval runtime
- protocol-first retrieval service
- platform-scale retrieval server

That split matters because each one creates a different maintenance burden.

- In app-coupled retrieval runtimes, the main risk is frontend/backend contract drift.
- In app-coupled retrieval runtimes, config-editing surfaces can amplify that drift when "save and apply" is not backed by one shared runtime state model.
- In protocol-first retrieval services, the main risk is bootstrap and packaging drift around the real protocol surface.
- In platform-scale retrieval servers, the main risk is combinatorial complexity across interfaces, bindings, and backends.

If the goal is a complete interactive app around retrieval, `deep-rag` is the clearest reference.

If the goal is a clean machine-to-machine retrieval surface, `MODULAR-RAG-MCP-SERVER` is the most relevant reference once you look past its launcher drift.

If the goal is a broad operational platform that can serve many usage modes from one core, `LightRAG` is the strongest example.

## Open Questions

- Can `deep-rag` reduce frontend/backend coupling in ReAct mode without losing its current UI clarity?
- Can `deep-rag` make its config editor truthful by centralizing hot-reload propagation instead of refreshing only part of the runtime?
- Can `MODULAR-RAG-MCP-SERVER` make the real MCP serving path the default user path?
- Can `LightRAG` keep its serving flexibility without making the effective health of each surface too hard to reason about?
