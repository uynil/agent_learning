# Agents Research Execution Rules

这份文档只负责说明学习系统本身的执行规则，不再重复仓库介绍或完整流程。

它记录的是：

- 学习系统如何运作
- 文档如何分工
- 什么时候更新哪些系统文档
- 哪些经验可以晋升为学习系统规则

它不记录的是：

- 某个 agent 项目的具体架构经验
- 某类 orchestration 的设计结论
- 某个项目里的 prompt 技巧总结

这些内容应进入 `docs/research/*`，而不是进入 `AGENTS.md`。

## Canonical Sources

如果多个文档说法不同，按下面顺序裁决：

1. [learning-workflow.md](/Users/uynil/projects/agents/docs/system/workflow/learning-workflow.md)
   研究流程唯一真相源
2. [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
   执行规则、更新边界、经验晋升规则
3. [prompt-library.md](/Users/uynil/projects/agents/docs/system/prompts/prompt-library.md)
   可直接运行的 prompts
4. [README.md](/Users/uynil/projects/agents/README.md)
   人类入口说明，不作为流程裁决依据

## Document Ownership

- [README.md](/Users/uynil/projects/agents/README.md)
  只回答“这是什么”和“从哪里开始”

- [learning-workflow.md](/Users/uynil/projects/agents/docs/system/workflow/learning-workflow.md)
  只维护完整研究流程、完成标准和收尾检查

- [prompt-library.md](/Users/uynil/projects/agents/docs/system/prompts/prompt-library.md)
  只维护可执行的 prompt 套件

- `docs/system/*`
  只维护学习系统本身：流程、prompts、评估、方法、模板

- `docs/system/methodology/*`
  只维护原则、检查清单、分析视角，不重复 prompt 正文

- `docs/research/projects/<project>/README.md`
  该项目的唯一入口页和当前状态页

- `docs/research/*`
  记录 agent 项目本身的设计、架构、调度、prompt 经验

## Project Notes Rules

每个项目目录只保留一个总入口页：

- `docs/research/projects/<project>/README.md`

如果需要更深入的内容，再补：

- `architecture.md`
- `llm-orchestration.md`
- `context-management.md`
- `deep-research-mode.md`
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

- `docs/research/projects/<project>/README.md`
- 需要时的项目专项文档
- `docs/system/prompts/prompt-iteration-log.md`

### Low-Frequency Updates

只有达到门槛时才更新：

- [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
- `docs/research/comparisons/*`
- [prompt-smoke-cases.md](/Users/uynil/projects/agents/docs/system/prompts/prompt-smoke-cases.md)
- [research-index.md](/Users/uynil/projects/agents/docs/system/workflow/research-index.md)

其中：

- `AGENTS.md` 只更新学习系统本身的规则经验
- `docs/research/comparisons/*` 更新 agent 设计和架构层面的跨项目经验

## Promotion Rules for AGENTS.md

只有“学习系统本身”的经验才允许晋升到 `AGENTS.md`。

例如：

- 哪种研究流程更稳
- 哪种收尾规则能减少漂移
- 哪种 prompt 使用方式更适合这个学习系统
- 哪种文档边界能降低维护成本

agent 项目本身的设计经验不进入 `AGENTS.md`，而应进入：

- `docs/research/projects/<project>/README.md`
- `docs/research/projects/<project>/*.md`
- `docs/research/comparisons/*`

在满足上面的内容边界之后，经验还需要满足下面至少一条，才允许晋升到 `AGENTS.md`：

- 在两个以上项目中重复出现
- 在连续两轮研究中都被证明有效

其他经验先留在：

- `docs/system/prompts/prompt-iteration-log.md`
- `docs/research/projects/<project>/notes.md`

## Blocked or Low-Confidence Research Rule

研究失败也必须留下记录，不允许直接丢失。

最低记录要求：

- `status: blocked` 或 `status: low-confidence`
- 已查看文件
- 当前阻塞点
- 当前假设
- 下次继续的起点

建议落在：

- `docs/research/projects/<project>/README.md`
- 或 `docs/research/projects/<project>/notes.md`

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

每轮研究结束时，按 [session-close-checklist.md](/Users/uynil/projects/agents/docs/system/workflow/session-close-checklist.md) 收尾。

不要只凭感觉判断“这一轮差不多结束了”。

同时要区分两类沉淀去向：

- 学习系统规则经验 -> `AGENTS.md`
- agent 设计 / 架构 / prompt 研究经验 -> `docs/research/*`

## Maintenance Principles

- 单一职责优先，避免同一条规则在多个文件里写完整版本
- 优先补充事实，再改结论
- 优先记录低置信度和失败路径，而不是假装研究顺利完成
- 让高频路径尽量轻，让低频沉淀有明确门槛
