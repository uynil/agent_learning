# Project Notes

This folder stores project-specific research outputs.

It does not store workflow rules, prompt library rules, or system-level evaluation criteria. Those belong in [docs/system](/Users/uynil/projects/agents/docs/system).

Each project folder uses `README.md` as its canonical entry page.

Additional files are optional and should only be created when the research depth requires them:

- `README.md`
- `architecture.md`
- `llm-orchestration.md`
- `context-management.md`
- `deep-research-mode.md`
- `prompts-analysis.md`
- `notes.md`

## Current Project Status

| Project | Status | Next Action |
| --- | --- | --- |
| BettaFish | first-pass complete | validate forum loop and report-generation behavior with one live run |
| DeepSearchAgent-Demo | first-pass complete | validate live run and compare with other research agents |
| MiroFish | partially live-validated | graph/build/prepare plus single-platform interview/close-env are validated; next run one full ReportAgent pass against a warm simulation and inspect readiness/log drift inside the report loop |
| Multi-Agent-Data-Analyst | first-pass complete | validate one SQL and one DataFrame run and confirm README/runtime alignment |
| MultiAgent-Data-Analyst | first-pass complete | pick the canonical entry path and repair memory/model gaps before live run |
| MODULAR-RAG-MCP-SERVER | partially live-validated | compare its file-backed modular RAG and MCP pattern against LightRAG and deep-rag; only revisit runtime if a clean Chroma-backed query is needed |
| LightRAG | partially live-validated | compare its graph-plus-vector retrieval topology against `deep-rag` and `MODULAR-RAG-MCP-SERVER`; only revisit server or WebUI if serving-layer evidence is needed |
| autoresearch | in-progress | validate one real experiment loop and compare against other research-system shapes |
| chatwoot | not-started | quick scan |
| deep-rag | first-pass complete | compare its knowledge-map navigation pattern with graph-heavy RAG systems and only revisit real-provider tests if needed |
| everything-claude-code | not-started | quick scan |
| gpt-researcher | first-pass complete | validate live run and compare against deep-research samples |
| rag-skill | in-progress | validate a true agent-invoked skill run and compare it with the cold-start traces |
| self-evolving-agent | in-progress | trace protocol split across skill, coordinator, hooks, ledgers, and benchmarks |
