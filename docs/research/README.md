# Agent Research Docs

这个目录只放“agent 项目研究产出”，不放学习系统本身的流程和规则。

## 这里放什么

- `projects/`
  每个 agent 项目一组文档，`README.md` 是该项目的唯一入口页
- `comparisons/`
  跨项目的比较和归纳

## 这里不放什么

- 学习流程规则
- prompt 套件设计规则
- 评估 rubrics
- session-close checklist

这些内容应进入 [docs/system](/Users/uynil/projects/agents/docs/system)。

## 使用方式

1. 先在 `projects/<project>/README.md` 记录项目级结论
2. 需要更深时，再补 `architecture.md`、`llm-orchestration.md`、`prompts-analysis.md`、`notes.md`
3. 当两个以上项目出现稳定共性，再把结论抽到 `comparisons/`
