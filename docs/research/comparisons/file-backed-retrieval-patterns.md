# File-Backed Retrieval Patterns

## Comparison Goal

Compare how projects implement local knowledge retrieval when they do not rely on a classic vector-store RAG pipeline in the main path.

## Compared Projects

- [rag-skill](/Users/uynil/projects/agents/docs/research/projects/rag-skill/README.md)
- [deep-rag](/Users/uynil/projects/agents/docs/research/projects/deep-rag/README.md)

## Dimensions

- metadata source
- retrieval contract
- runtime control plane
- granularity of retrieval
- compression strategy
- environment assumptions
- dominant failure mode

## Comparison Table

| Dimension | rag-skill | deep-rag |
| --- | --- | --- |
| Main style | prompt-governed local retrieval workflow | coded tool-using local RAG runtime |
| Metadata source | manually maintained `data_structure.md` files plus file-type reference docs | generated knowledge map from `summary.txt` plus optional chunk artifacts |
| Retrieval contract | navigate indexes, learn file-type handling, then use shell tools or Python libs to inspect local files | inject file-summary map into the system prompt, then let the model call `retrieve_files` |
| Control plane | mostly in `SKILL.md` and reference docs | mostly in backend code plus system prompts |
| Retrieval granularity | file-type-specific and locally selective: grep hits, nearby lines, sheet previews, filtered rows | path-based retrieval of files, directories, or `/`, with raw observations returned to the loop |
| Compression timing | mostly before and during retrieval: narrow with index docs, grep, offsets, `nrows`, and local previews | mostly before retrieval via summary generation; little explicit compression after runtime retrieval |
| Environment dependency shape | high dependence on shell and interpreter details in the target working directory | higher dependence on the app/runtime environment, but the retrieval contract is centralized in code |
| Main strengths | cheap to adopt, transparent workflow, good token discipline on curated corpora | cleaner runtime contract, generated metadata, explicit retrieval tool, less manual index maintenance |
| Main weaknesses | index drift, corpus drift, tool availability drift, `python` vs `python3` ambiguity | risk of oversized retrieval payloads, weak post-retrieval compression, config/runtime mismatch risk |

## Metadata Lifecycle

| Question | rag-skill | deep-rag |
| --- | --- | --- |
| Where does retrieval metadata come from? | humans write `data_structure.md` by hand | preprocessing generates `summary.txt` from the corpus |
| How does metadata evolve? | manual edits whenever files or folders change | rerun summary generator when corpus changes |
| Main maintenance burden | doc freshness and corpus/index alignment | generator quality and regeneration discipline |
| Main drift risk | index claims no longer match actual files or actual file contents | generated summaries become stale or over-compressed |
| What did live validation show? | safety index over-claimed defensive coverage in `XSS.md` | summary loading worked cleanly at runtime and produced a 2711-character knowledge map |

## Retrieval Envelope

| Question | rag-skill | deep-rag |
| --- | --- | --- |
| How is retrieval narrowed? | by index docs, grep hits, local previews, file-type-specific steps | by summary-guided path selection, then raw file or directory retrieval |
| Typical retrieval unit | nearby lines, one file, one sheet preview, filtered rows | exact file, exact directory, or entire corpus |
| Post-retrieval compression | mostly manual and local during retrieval | little to none after retrieval in the runtime loop |
| Runtime evidence | Excel path stayed narrow and explainable; PDF path needed local extraction before grep | single file ~22.9 KB, one directory ~86.5 KB, full corpus ~449 KB |
| Main risk | retrieval may fail before content is found if tools or local setup drift | retrieval may succeed too broadly and flood the loop with raw text |

## Environment Coupling

| Question | rag-skill | deep-rag |
| --- | --- | --- |
| What part is most environment-sensitive? | shell tools and interpreter aliases in the target working directory | backend dependency setup, provider credentials, and env variable naming |
| Concrete runtime issue found | `python` and `python3` resolved to different interpreters, changing package availability | app boots after local setup, but shipped API keys are placeholders and documented path vars are ignored |
| Failure shape | agent may choose the wrong interpreter and fail before retrieval | app reaches retrieval fine, but `/chat` stops at provider boundary or uses unintended default paths |
| Recovery shape | probe tools and interpreters in-project before handling PDF/Excel | create venv, validate endpoints first, then fix credentials and env names |

## Runtime Validation Snapshot

| Runtime checkpoint | rag-skill | deep-rag |
| --- | --- | --- |
| Runnable entrypoint in repo | no dedicated runner; only skill assets and corpus | yes; FastAPI backend and frontend scripts |
| Knowledge navigation validated | yes, via cold-start trace | yes, via live backend endpoints |
| Retrieval endpoint validated | manual simulation only | yes, `/knowledge-base/retrieve` |
| End-to-end chat validated | no dedicated host to validate | yes, with a local OpenAI-compatible stub in both function and ReAct modes |
| Real vendor-backed answer validated | no | no, shipped credentials are placeholders |
| Strongest validated claim | documentation-first retrieval can work on curated local files | knowledge-map-driven runtime retrieval and dual-mode orchestration both work end-to-end when the provider matches the expected streaming contract |

## Shared Patterns

- both projects avoid a classic embedding-plus-vector-database retrieval layer in their main path
- both use a human-readable metadata layer before direct content retrieval happens
- both rely on active retrieval decisions instead of assuming one precomputed ranking step is enough
- both are better described as file-backed retrieval systems than as "standard RAG"
- both show that retrieval quality depends heavily on the quality of the metadata layer that guides search

## Key Differences

- `rag-skill` uses manually authored metadata, while `deep-rag` generates its knowledge map from the corpus
- `rag-skill` puts much of the retrieval logic into prompt instructions and local tool habits, while `deep-rag` puts it into a dedicated runtime and tool schema
- `rag-skill` narrows aggressively before reading content, while `deep-rag` narrows through metadata first but may still return very large raw payloads once a path is selected
- `rag-skill` is more fragile to local environment ambiguity, especially shell alias and interpreter differences
- `deep-rag` is more stable as a runtime shape, but more exposed to context blow-up once retrieval scope becomes too broad
- `deep-rag` has a cleaner function-mode client contract, but its ReAct fallback currently leaks protocol markers directly into the visible answer stream
- `rag-skill` treats file-type handling docs as part of the control plane, while `deep-rag` treats preprocessing summaries and runtime prompts as the main control plane

## Conclusions

These two projects suggest a useful split inside non-vector local retrieval systems:

- manual-index workflow retrieval
- generated-knowledge-map runtime retrieval

That split matters because the systems fail differently.

- In manual-index workflow retrieval, the main risk is operational drift:
  - stale index docs
  - incomplete corpus docs
  - missing local tools
  - interpreter ambiguity
- In generated-knowledge-map runtime retrieval, the main risk is retrieval payload management:
  - too much raw content
  - weak post-retrieval compression
  - oversized observations re-entering the loop

If the corpus is small, curated, and close to the operator, `rag-skill` shows that a documentation-first retrieval workflow can be surprisingly effective.

If the goal is a reusable runtime with a cleaner retrieval interface, `deep-rag` is the stronger pattern because it turns retrieval into an explicit tool contract and reduces dependence on hand-maintained indexes.

If the goal is a lightweight, portable skill that can adapt to mixed local file types without a backend service, `rag-skill` is the more direct pattern, but only if the operator also controls the working environment closely enough to manage interpreter and tool drift.

## Open Questions

- At what corpus size does generated metadata become clearly superior to manual indexes?
- Can `deep-rag` keep its retrieval contract clean while adding a post-retrieval compression stage?
- Can `rag-skill` stay lightweight while adding environment-detection guidance and index-drift checks?
- Is there a hybrid pattern where generated metadata handles navigation while file-type reference docs still handle specialized local processing?
