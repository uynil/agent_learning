# Agents Research Workspace Guide

这个仓库用于研究和理解开源 `agents` 项目，重点关注：

- 架构设计
- LLM 调度设计
- prompts 的组织、编写与使用
- 不同项目之间的模式对比
- 自身学习文档的持续迭代

当前仓库已经把示例项目放在 `examples/` 下，把研究文档系统放在 `docs/` 下。后续的研究、沉淀、复盘都优先围绕这套文档系统展开。

## 目录结构

### 代码样例

- `examples/`
  存放被研究的开源项目。每个子目录对应一个项目，例如 `gpt-researcher`、`DeepSearchAgent-Demo`、`MiroFish` 等。

### 文档系统

- `docs/README.md`
  文档系统总入口。先读这里，了解整体结构。

- `docs/00-system/learning-workflow.md`
  研究流程说明。定义了 `Quick Scan`、`Deep Dive`、`Cross Project Compare`、`Prompt Audit`、`Doc Refine` 这几种模式。

- `docs/00-system/prompt-library.md`
  可直接复制给 LLM 使用的 prompts，包括总控 prompt、项目快扫 prompt、架构分析 prompt、LLM 调度分析 prompt、prompt 审计 prompt、文档整理 prompt。

- `docs/00-system/evaluation-rubric.md`
  研究输出质量的评分标准。每轮研究后可按这个标准复盘。

- `docs/00-system/prompt-iteration-log.md`
  prompt 迭代记录。记录哪些 prompt 好用、哪里太空泛、下轮怎么改。

- `docs/01-projects/`
  单项目研究目录。每个项目一个子目录，用来存放项目自己的研究文档。

- `docs/02-comparisons/`
  横向对比目录。用于沉淀多个项目之间的共同模式与差异。

- `docs/03-methodology/`
  方法论文档。定义“如何读一个 agent 项目”“如何分析 prompts”“如何分析 orchestration”。

- `docs/templates/`
  模板目录。创建新研究文档时优先从这里复制模板。

## 推荐使用顺序

每次开始研究时，按下面顺序操作：

1. 先读 `docs/README.md`
2. 再读 `docs/00-system/learning-workflow.md`
3. 从 `docs/00-system/prompt-library.md` 里选择本轮要用的 prompt
4. 在 `examples/<project>/` 中阅读目标项目
5. 把研究结果写入 `docs/01-projects/<project>/`
6. 如果出现跨项目模式，再同步到 `docs/02-comparisons/`
7. 总结本轮经验，必要时更新 `AGENTS.md`
8. 最后更新 `docs/00-system/prompt-iteration-log.md`

## 单项目研究流程

### 第一步：Quick Scan

目的：

- 快速判断项目解决什么问题
- 确定入口文件、核心 agent、核心 prompts、state、tools
- 决定是否值得继续深挖

使用：

- 复制 `docs/00-system/prompt-library.md` 中的 `Master Prompt`
- 再复制 `Quick Scan Prompt`

输出去向：

- 先写入 `docs/01-projects/<project>/project-overview.md`
  如果文件还不存在，就从 `docs/templates/project-overview-template.md` 复制一份

### 第二步：Deep Dive

如果项目值得继续研究，再做深入分析：

- 架构设计：使用 `Architecture Prompt`
- LLM 调度：使用 `LLM Orchestration Prompt`
- prompt 设计：使用 `Prompt Audit Prompt`

输出去向：

- `docs/01-projects/<project>/architecture.md`
- `docs/01-projects/<project>/llm-orchestration.md`
- `docs/01-projects/<project>/prompts-analysis.md`
- `docs/01-projects/<project>/notes.md`

创建这些文件时，优先使用：

- `docs/templates/architecture-template.md`
- `docs/templates/llm-orchestration-template.md`
- `docs/templates/prompts-analysis-template.md`
- `docs/templates/notes-template.md`

### 第三步：Doc Refine

完成一轮研究后，用 `Doc Refine Prompt` 把分散输出整理成可持续更新的文档。

目标：

- 把聊天式输出变成笔记式输出
- 让后续补充内容时不需要重写整份文档
- 明确“已确认事实”和“尚未确认问题”

## 横向对比的使用方式

当至少两个项目有初步研究结果后，再开始更新 `docs/02-comparisons/` 下的文档。

当前已预留的主题包括：

- `prompt-organization-patterns.md`
- `llm-scheduling-patterns.md`
- `state-management-patterns.md`
- `tool-use-patterns.md`

使用原则：

- 一份对比文档只关注一个维度
- 先填表格，再写结论
- 结论必须尽量建立在单项目文档之上，避免直接凭印象总结

## 更新文档时的规则

所有研究文档都应尽量区分下面四类内容：

- `Observed Facts`
  代码或文档中可以直接确认的事实

- `Inferences`
  基于事实得出的理解和推断

- `Open Questions`
  目前还没确认、需要下轮继续看的问题

- `Improvement Ideas`
  未来可以继续研究、抽象、优化的点

不要把这四类内容混在一起写。

## 每轮研究后的收尾动作

每完成一轮研究，除了更新项目文档外，还要额外做两件事：

1. 总结本轮经验
2. 视情况更新这份 `AGENTS.md`

这里的“经验”包括但不限于：

- 哪种研究顺序更顺手
- 哪类 prompt 更有效
- 哪些文档模板不够用
- 哪些对比维度值得固定下来
- 哪些常见误区需要提前提醒

更新 `AGENTS.md` 的原则：

- 只沉淀可复用的经验，不记录一次性的临时细节
- 优先更新方法、流程、规范、注意事项
- 如果某条经验已经稳定出现两次以上，应尽量写进 `AGENTS.md`
- 让 `AGENTS.md` 成为这套研究系统的“活说明书”

## Frontmatter 规则

项目文档应尽量保留并更新头部元信息，例如：

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

每次更新文档时，至少同步更新：

- `status`
- `last_updated`
- `next_action`

## Prompt 的使用建议

### 推荐做法

- 先用 `Master Prompt` 让模型判断当前模式
- 再用对应的专项 prompt 深挖
- 输出后手动整理到文档，而不是直接把聊天记录原样粘进去
- 每轮研究后总结哪些方法有效，并回写到 `AGENTS.md`
- 每轮研究后记录 prompt 的好坏

### 不推荐做法

- 一开始就让模型“全面分析整个项目”
- 不区分事实和推断
- 只做聊天总结，不写入文档
- 每次都重新发明 prompt，不积累迭代记录

## 开启一轮新研究时的最小操作

如果只是想快速开始，使用下面这套最小流程：

1. 选一个 `examples/<project>`
2. 打开 `docs/00-system/prompt-library.md`
3. 使用 `Master Prompt`
4. 使用 `Quick Scan Prompt`
5. 把结果整理到 `docs/01-projects/<project>/project-overview.md`
6. 更新 `prompt-iteration-log.md`

## 当前阶段的边界

当前文档系统已经初始化完成，但大部分项目研究内容还没有开始。

因此：

- `docs/01-projects/` 下的大多数项目目录目前只是占位
- `docs/02-comparisons/` 下的内容目前主要是骨架
- 后续应优先补单项目研究，再补横向对比

## 建议的起步顺序

如果没有特别指定，建议先从这些项目开始：

1. `examples/DeepSearchAgent-Demo`
2. `examples/gpt-researcher`
3. `examples/BettaFish`
4. `examples/MiroFish`
5. 其他项目

原因：

- `DeepSearchAgent-Demo` 结构相对清晰，适合建立基本研究方法
- `gpt-researcher` 适合理解 planner / execution 风格
- `BettaFish` 和 `MiroFish` 适合研究更复杂的调度和系统演化

## 维护原则

- 优先最小增量更新，不重写已有内容
- 优先补充事实，再修改结论
- 每次只解决一个研究问题
- 对不确定的内容明确标注，而不是假装确定
- 让文档越来越适合复用，而不是越来越像聊天记录
- 每轮研究结束后，把稳定有效的经验沉淀到 `AGENTS.md`
