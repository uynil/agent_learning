# Multi-Agent Data Analyst Variants

## Comparison Goal

Compare the two similarly named repositories:

- [Multi-Agent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/Multi-Agent-Data-Analyst/README.md)
- [MultiAgent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/MultiAgent-Data-Analyst/README.md)

The names are close enough that it is easy to assume they are variants of the same architecture. The code suggests they are actually useful as a contrast pair.

## Compared Projects

- [Multi-Agent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/Multi-Agent-Data-Analyst/README.md)
- [MultiAgent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/MultiAgent-Data-Analyst/README.md)

## Dimensions

- canonical runtime shape
- what "multi-agent" means in practice
- LLM involvement
- prompt organization
- context and memory model
- deep-research behavior
- main risks
- best learning value

## Comparison Table

| Dimension | Multi-Agent-Data-Analyst | MultiAgent-Data-Analyst |
| --- | --- | --- |
| Canonical runtime shape | single bounded runtime around `main.py` + `app/agent_orchestrator.py` | broader workflow spread across `src/orchestrator.py`, `streamlit_app/*`, and `mcp/manifest.json` |
| Practical agent pattern | one orchestrator that delegates to SQL or visualization specialists | staged pipeline of profiler, EDA, model, verifier, and notebook agents |
| What "multi-agent" really means | route-and-delegate tool orchestration | modular workflow handoff via A2A events and stored artifacts |
| LLM involvement | high; multiple calls per request and executable code generation | lower; core pipeline is deterministic, Gemini is used mainly for narrative explanation |
| Prompt organization | inline prompts tightly coupled to code execution | sparse embedded prompts mainly for report writing and explanation |
| Prompt-to-code coupling | very tight; model output is often executed | loose; model output is displayed or embedded into notebooks |
| Context model | request-scoped context, schema/sample injection, little persistent memory | `st.session_state` + JSON memory + A2A messages + artifact files |
| Deep-research behavior | weak, but it does have bounded retries and stepwise subpipelines | absent; this is not a query decomposition or reflective research system |
| Main risks | prompt/code safety, `exec()` on generated code, stale README/runtime mismatch | incomplete model path, broken memory API usage, fragmented entry surfaces |
| Best study value | prompt-to-code coupling, typed orchestration, local retry loops | workflow decomposition, A2A messaging, resumability design, and glue-layer failure modes |

## Shared Patterns

- Both projects call themselves "multi-agent," but neither is a deep-research planner/reviewer system.
- Both are built for data analysis rather than open-ended assistant chat.
- Both rely on specialized modules instead of one giant prompt.
- Both still need live-run validation before their runtime behavior can be treated as fully confirmed.

## Key Differences

- [Multi-Agent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/Multi-Agent-Data-Analyst/README.md) is more coherent as a single agent runtime, even if its "multi-agent" claim is somewhat overstated.
- [MultiAgent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/MultiAgent-Data-Analyst/README.md) has a wider product surface, but the architecture is looser and the glue code is less reliable.
- The first project places more intelligence in LLM orchestration; the second places more intelligence in the workflow structure and local analytics code.
- The first project is more useful for studying prompt structure and LLM scheduling; the second is more useful for studying context handoff, memory design, and workflow failure modes.

## Tentative Conclusions

These two repositories should not be grouped into the same learning bucket.

- Study [Multi-Agent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/Multi-Agent-Data-Analyst/README.md) when the question is: how does an LLM-centric analysis orchestrator route work, constrain outputs, and survive runtime errors?
- Study [MultiAgent-Data-Analyst](/Users/uynil/projects/agents/docs/research/projects/MultiAgent-Data-Analyst/README.md) when the question is: how do agent stages hand off artifacts, coordinate with a bus, and attempt resumability in a data workflow?

The similar names are misleading; architecturally they are closer to adjacent examples than to minor forks.

## Open Questions

- After live runs, will `Multi-Agent-Data-Analyst` still look like a coherent single-runtime example?
- After repairs, will `MultiAgent-Data-Analyst` behave like a true autonomous chain, or mainly as a manually assisted demo pipeline?
- Should the second project be compared more with workflow systems than with LLM-orchestrator agents?
