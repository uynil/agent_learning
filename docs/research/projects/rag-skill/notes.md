---
project: rag-skill
source_path: examples/rag-skill
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: compare these cold-start traces with a true agent-invoked skill session and check whether the agent discovers the `python` vs `python3` split on its own
---

# rag-skill Notes

## Purpose

This note records a cold-start execution trace that is closer to how an agent would experience `examples/rag-skill` from inside the project directory itself.

The goal here is not to prove answer quality in the abstract. The goal is to verify:

- what commands are naturally available in the target working directory
- whether the documented pre-read gates are operationally usable
- where the workflow succeeds or fails without extra manual help

## Runtime Entry Check

### Observed Facts

- The example repository contains no runnable skill host, CLI wrapper, or app entrypoint.
- The repository contents are primarily:
  - `.agent/skills/rag-skill/SKILL.md`
  - reference docs
  - sample knowledge files
- This means the closest available validation method is a cold-start trace that manually follows `SKILL.md`.

### Inference

- The example is a transferable skill package, not a self-hosted runtime demo.

## Environment Probe In Project Working Directory

Working directory used for this trace:

```text
/Users/uynil/projects/agents/examples/rag-skill
```

### Command Resolution

```bash
which python
which python3
python -V
python3 -V
```

### Observed Facts

- `python` resolves to `/Users/uynil/miniconda3/bin/python`
- `python3` resolves to `/opt/homebrew/bin/python3`
- `python` is Python 3.10.8
- `python3` is Python 3.11.6

### Package Availability

Under `python`:

- `pandas`: available
- `openpyxl`: available
- `fitz`: available
- `pdfplumber`: missing
- `pypdf`: missing

Under `python3`:

- `pandas`: missing
- `openpyxl`: missing
- `fitz`: missing
- `pdfplumber`: missing
- `pypdf`: missing

### Inferences

- The project's practical behavior depends heavily on whether an agent chooses `python` or `python3`.
- The current skill docs describe libraries and workflows, but they do not warn that interpreter aliases may resolve to materially different environments.
- In this working directory, a naive `python3` choice breaks both the PDF and Excel paths.

## Trace 1: Markdown Query

Query:

```text
XSS 攻击的防护措施有哪些？
```

### Steps

1. Confirm the default knowledge root:

```bash
test -d knowledge
```

2. Read the root index:

```bash
sed -n '1,120p' knowledge/data_structure.md
```

3. Read the domain index:

```bash
sed -n '1,80p' 'knowledge/Safety Knowledge/data_structure.md'
```

4. Search inside the candidate file:

```bash
rg -n "防护|防御|HttpOnly|CSP|转义|过滤|innerHTML" 'knowledge/Safety Knowledge/XSS.md'
```

### Observed Facts

- The root index correctly routes the query to `Safety Knowledge/`.
- The safety index claims `XSS.md` covers:
  - 原理
  - 类型
  - 常见利用方式
  - 防御策略
- The actual grep hits are thin for a defense-focused question:
  - `innerHTML` appears in the DOM-based XSS explanation
  - `HttpOnly` appears as a mitigation against cookie theft
  - there is no strong complete defense section surfaced by direct retrieval

### Inference

- The retrieval workflow itself works, but the corpus/index contract is overstated for this question.

## Trace 2: PDF Query

Query:

```text
2026年AI Agent技术有哪些关键发展趋势？
```

### Steps

1. Confirm the root and read the AI domain index:

```bash
sed -n '1,80p' 'knowledge/AI Knowledge/data_structure.md'
```

2. Follow the documented pre-read gate:

```bash
sed -n '1,120p' '.agent/skills/rag-skill/references/pdf_reading.md'
```

3. Probe the recommended tools from the actual project working directory:

```bash
command -v pdftotext
python3 -c "import pdfplumber, pypdf, fitz"
python -c "import fitz"
```

4. Try the practical fallback that actually exists in this environment:

```bash
python - <<'PY'
import fitz
from pathlib import Path
pdf = Path('knowledge/AI Knowledge/2026年AI Agent智能体技术发展报告.pdf')
out = Path('/tmp/rag_skill_ai_agent_2026_conda.txt')
doc = fitz.open(pdf)
with out.open('w', encoding='utf-8') as f:
    for i, page in enumerate(doc):
        f.write(f'\\n===== PAGE {i+1} =====\\n')
        f.write(page.get_text('text'))
print(out, doc.page_count)
PY
```

5. Search the extracted text:

```bash
rg -n "多智能体|MCP|A2A|反思|记忆模块|未来发展趋势" /tmp/rag_skill_ai_agent_2026_conda.txt
```

### Observed Facts

- The AI index clearly points to `2026年AI Agent智能体技术发展报告.pdf`.
- The documented preferred tools are not available by default:
  - `pdftotext` missing
  - `pdfplumber` missing
  - `pypdf` missing
- `python3` cannot import `fitz` in the project working directory.
- `python` can import `fitz` and can extract the PDF.
- The extracted report supports several concrete trend points:
  - multi-agent systems becoming mainstream
  - MCP and A2A acting as interoperability foundations
  - reflection and self-critique becoming a stronger agent loop
  - memory being treated as a first-class architectural module
  - future framing around general agents, embodied AI, edge agents, and the agent internet

### Inferences

- The PDF flow is viable only if the agent either:
  - picks `python` instead of `python3`, or
  - explicitly discovers another valid interpreter path
- The skill's conceptual workflow is solid, but the operational path is brittle because the recommended tool stack is underspecified relative to real shell environments.

## Trace 3: Excel Query

Query:

```text
帮我分析一下库存数据，哪些商品库存不足？
```

### Steps

1. Read the e-commerce domain index:

```bash
sed -n '1,80p' 'knowledge/E-commerce Data/data_structure.md'
```

2. Follow the documented pre-read gate:

```bash
sed -n '1,60p' '.agent/skills/rag-skill/references/excel_reading.md'
sed -n '1,60p' '.agent/skills/rag-skill/references/excel_analysis.md'
```

3. Probe the interpreter path:

```bash
python3 -c "import pandas"
python -c "import pandas, openpyxl"
```

4. Preview the workbook with the interpreter that actually works:

```bash
python - <<'PY'
import pandas as pd
df = pd.read_excel('knowledge/E-commerce Data/inventory.xlsx', nrows=5)
print(df.to_string(index=False))
PY
```

5. Run the low-stock filter:

```bash
python - <<'PY'
import pandas as pd
df = pd.read_excel('knowledge/E-commerce Data/inventory.xlsx')
low = df[df['stock_on_hand'] <= df['reorder_level']].copy()
low['gap'] = low['stock_on_hand'] - low['reorder_level']
print('TOTAL_ROWS=', len(df))
print('LOW_ROWS=', len(low))
print(low.sort_values('gap')[['sku','product','category','warehouse','stock_on_hand','reorder_level','gap']].head(8).to_string(index=False))
PY
```

### Observed Facts

- The e-commerce index correctly points to `inventory.xlsx`.
- The index text says the directory is "当前以 CSV 为主", but the actual files are Excel workbooks.
- `python3` cannot import `pandas` in this working directory.
- `python` can import `pandas` and `openpyxl`.
- The workbook is straightforward:
  - 1 sheet
  - 200 rows
  - 8 columns
- Using `stock_on_hand <= reorder_level`, the workbook contains 33 low-stock rows.

### Inferences

- The Excel analysis path is strong at the data level but still depends on interpreter choice at runtime.
- Relative to the PDF path, Excel is less semantically ambiguous because the schema directly exposes the answer surface.

## Cross-Trace Summary

## Observed Facts

- The index navigation pattern works well across all three file types.
- The mandatory pre-read rule is easy to follow manually because the reference docs are local and concise.
- The project working directory does not expose a single stable Python environment through `python3`.
- The default shell experience inside the project is good enough for text search, but not good enough for the documented Python-heavy workflows unless the agent picks the right interpreter.

## Inferences

- The most important runtime risk is not retrieval logic alone. It is environment ambiguity.
- `rag-skill` currently teaches "what to do" better than "how to detect the environment you are actually in before doing it".

## Open Questions

- In a real agent session, would the agent naturally probe both `python` and `python3`, or would it fail early on the first missing import?
- Should the skill explicitly recommend an interpreter probe step before any PDF or Excel handling?
- Would a cached-text-first workflow be more robust than library-based extraction for this kind of portable skill demo?
