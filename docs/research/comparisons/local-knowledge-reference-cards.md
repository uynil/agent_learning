# Local-Knowledge Reference Cards

## Goal

Provide one compact reference card per project so the broader comparison set is easier to scan without reopening every project README.

## rag-skill

- Shape: documentation-governed skill workflow over a local mixed-format corpus
- Best at: transparent, low-infrastructure retrieval with human-visible procedure
- Prompt identity: skill docs and reference docs are the control plane
- Context identity: narrow, procedural local reads guided by `data_structure.md`
- Loop identity: protocol loop with up to 5 rounds
- Session identity: artifact-first and human-steered
- Main risk: index drift, environment drift, and tool/interpreter ambiguity

## deep-rag

- Shape: app-shaped file-navigation RAG with bounded function/ReAct loops
- Best at: visible query-time retrieval continuation inside one answer flow
- Prompt identity: model-visible knowledge-map navigation rules
- Context identity: summary injection plus raw observation replay
- Loop identity: bounded model-directed retrieval loop
- Session identity: client-replayed interactive session
- Main risk: raw retrieval payload growth, config drift, and protocol coupling

## MODULAR-RAG-MCP-SERVER

- Shape: modular retrieval core with MCP-facing service surface
- Best at: ranked chunk retrieval and tool-style service exposure
- Prompt identity: prompt-light, code- and protocol-dominant
- Context identity: request-scoped retrieval results plus trace artifacts
- Loop identity: one-call retrieval service rather than visible agent loop
- Session identity: caller-steered service continuity over persisted collections and traces
- Main risk: bootstrap drift, launcher drift, and abstraction leaks at the tool edge

## LightRAG

- Shape: graph-plus-vector retrieval platform with broad serving and storage surface
- Best at: persistent retrieval substrate with multiple modes and fallback branches
- Prompt identity: prompt-layered internal retrieval substrate shaping
- Context identity: token-budgeted graph/chunk context assembled from many stores
- Loop identity: pipeline-directed retrieval flow
- Session identity: substrate-backed workspace continuity
- Main risk: operational complexity across modes, stores, and serving layers

## Quick Sorting Guide

| If you want... | Start with... |
| --- | --- |
| the lightest workflow and most visible procedure | `rag-skill` |
| the most visible in-request retrieval continuation | `deep-rag` |
| the cleanest retrieval-service boundary | `MODULAR-RAG-MCP-SERVER` |
| the richest persistent retrieval substrate | `LightRAG` |

## Related Pages

- [local-rag-design-selection.md](/Users/uynil/projects/agents/docs/research/comparisons/local-rag-design-selection.md)
- [prompt-control-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/prompt-control-patterns.md)
- [local-knowledge-context-management-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/local-knowledge-context-management-patterns.md)
- [agent-loop-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/agent-loop-patterns.md)
- [session-management-patterns.md](/Users/uynil/projects/agents/docs/research/comparisons/session-management-patterns.md)
- [human-steering-vs-autonomous-continuity.md](/Users/uynil/projects/agents/docs/research/comparisons/human-steering-vs-autonomous-continuity.md)
