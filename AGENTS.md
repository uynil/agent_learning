# Agents Research Execution Rules

这份文档只负责说明执行规则，不再重复仓库介绍或完整流程。

## Canonical Sources

如果多个文档说法不同，按下面顺序裁决：

1. [learning-workflow.md](/Users/uynil/projects/agents/docs/00-system/learning-workflow.md)
   研究流程唯一真相源
2. [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
   执行规则、更新边界、经验晋升规则
3. [prompt-library.md](/Users/uynil/projects/agents/docs/00-system/prompt-library.md)
   可直接运行的 prompts
4. [README.md](/Users/uynil/projects/agents/README.md)
   人类入口说明，不作为流程裁决依据

## Document Ownership

- [README.md](/Users/uynil/projects/agents/README.md)
  只回答“这是什么”和“从哪里开始”

- [learning-workflow.md](/Users/uynil/projects/agents/docs/00-system/learning-workflow.md)
  只维护完整研究流程、完成标准和收尾检查

- [prompt-library.md](/Users/uynil/projects/agents/docs/00-system/prompt-library.md)
  只维护可执行的 prompt 套件

- `docs/03-methodology/*`
  只维护原则、检查清单、分析视角，不重复 prompt 正文

- `docs/01-projects/<project>/README.md`
  该项目的唯一入口页和当前状态页

## Project Notes Rules

每个项目目录只保留一个总入口页：

- `docs/01-projects/<project>/README.md`

如果需要更深入的内容，再补：

- `architecture.md`
- `llm-orchestration.md`
- `prompts-analysis.md`
- `notes.md`

不要再为同一个项目同时维护 `README.md` 和另一份独立的 overview 页面。

## Evidence Rules

所有研究结论尽量区分：

- `Observed Facts`
- `Inferences`
- `Open Questions`
- `Improvement Ideas`

不要把“看到的事实”和“自己的猜测”混写。

## High-Frequency vs Low-Frequency Updates

### High-Frequency Updates

每轮研究通常都要更新：

- `docs/01-projects/<project>/README.md`
- 需要时的项目专项文档
- `docs/00-system/prompt-iteration-log.md`

### Low-Frequency Updates

只有达到门槛时才更新：

- [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
- `docs/02-comparisons/*`
- [prompt-smoke-cases.md](/Users/uynil/projects/agents/docs/00-system/prompt-smoke-cases.md)
- [research-index.md](/Users/uynil/projects/agents/docs/00-system/research-index.md)

## Promotion Rules for AGENTS.md

只有满足下面至少一条，经验才允许晋升到 `AGENTS.md`：

- 在两个以上项目中重复出现
- 在连续两轮研究中都被证明有效

其他经验先留在：

- `docs/00-system/prompt-iteration-log.md`
- `docs/01-projects/<project>/notes.md`

## Blocked or Low-Confidence Research Rule

研究失败也必须留下记录，不允许直接丢失。

最低记录要求：

- `status: blocked` 或 `status: low-confidence`
- 已查看文件
- 当前阻塞点
- 当前假设
- 下次继续的起点

建议落在：

- `docs/01-projects/<project>/README.md`
- 或 `docs/01-projects/<project>/notes.md`

## Frontmatter Rules

项目入口页和专项文档尽量保留头部元信息，例如：

```yaml
---
project: DeepSearchAgent-Demo
source_path: examples/DeepSearchAgent-Demo
status: in-progress
confidence: medium
last_updated: 2026-03-21
next_action: inspect prompt execution flow
---
```

每次更新至少同步：

- `status`
- `last_updated`
- `next_action`

## Session Close Rule

每轮研究结束时，按 [session-close-checklist.md](/Users/uynil/projects/agents/docs/00-system/session-close-checklist.md) 收尾。

不要只凭感觉判断“这一轮差不多结束了”。

## Maintenance Principles

- 单一职责优先，避免同一条规则在多个文件里写完整版本
- 优先补充事实，再改结论
- 优先记录低置信度和失败路径，而不是假装研究顺利完成
- 让高频路径尽量轻，让低频沉淀有明确门槛
