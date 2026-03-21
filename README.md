# Agents Research Workspace

这是一个用于研究开源 `agents` 项目架构与设计的工作区。

这个仓库不只是收集代码样例，更重点在于沉淀一套可持续迭代的研究方法，帮助理解：

- agent 系统的架构设计
- LLM 调度与 orchestration 方式
- prompts 的组织、编写和调用方式
- 不同项目之间的共同模式与关键差异
- 自己的学习过程如何持续复盘和改进

## 这个仓库里有什么

### 1. 开源项目样例

所有样例项目都放在 [examples](/Users/uynil/projects/agents/examples) 下，用于实际阅读、拆解和对比。

当前已收录的项目包括：

- [BettaFish](/Users/uynil/projects/agents/examples/BettaFish)
- [DeepSearchAgent-Demo](/Users/uynil/projects/agents/examples/DeepSearchAgent-Demo)
- [MiroFish](/Users/uynil/projects/agents/examples/MiroFish)
- [autoresearch](/Users/uynil/projects/agents/examples/autoresearch)
- [chatwoot](/Users/uynil/projects/agents/examples/chatwoot)
- [everything-claude-code](/Users/uynil/projects/agents/examples/everything-claude-code)
- [gpt-researcher](/Users/uynil/projects/agents/examples/gpt-researcher)

### 2. 研究文档系统

研究文档都放在 [docs](/Users/uynil/projects/agents/docs) 下，已经初始化好目录和模板。

主要结构：

- [docs/00-system](/Users/uynil/projects/agents/docs/00-system)
  研究流程、prompt 库、评估标准、prompt 迭代日志
- [docs/01-projects](/Users/uynil/projects/agents/docs/01-projects)
  单项目研究文档
- [docs/02-comparisons](/Users/uynil/projects/agents/docs/02-comparisons)
  多项目横向对比
- [docs/03-methodology](/Users/uynil/projects/agents/docs/03-methodology)
  方法论与分析框架
- [docs/templates](/Users/uynil/projects/agents/docs/templates)
  可复用模板

### 3. 研究操作规则

根目录的 [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md) 是这套研究系统的“工作说明书”。

它重点说明：

- 这些文档各自怎么用
- 每轮研究按什么顺序推进
- 研究结果应该写到哪里
- 每轮研究后如何总结经验并回写系统文档

## 建议怎么开始

如果你是第一次进入这个仓库，建议按这个顺序：

1. 阅读 [docs/README.md](/Users/uynil/projects/agents/docs/README.md)
2. 阅读 [docs/00-system/learning-workflow.md](/Users/uynil/projects/agents/docs/00-system/learning-workflow.md)
3. 阅读 [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
4. 选择一个样例项目开始研究
5. 使用 [docs/00-system/prompt-library.md](/Users/uynil/projects/agents/docs/00-system/prompt-library.md) 中的 prompt
6. 把研究结果整理到对应的项目目录中

## 一轮研究通常怎么做

最小流程如下：

1. 在 [examples](/Users/uynil/projects/agents/examples) 中选一个项目
2. 用 `Master Prompt` 判断这轮研究模式
3. 先做 `Quick Scan`
4. 决定是否继续 `Deep Dive`
5. 把结果写到 [docs/01-projects](/Users/uynil/projects/agents/docs/01-projects) 对应目录
6. 如果发现跨项目共性，再更新 [docs/02-comparisons](/Users/uynil/projects/agents/docs/02-comparisons)
7. 总结本轮经验，并更新 [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md) 与 [prompt-iteration-log.md](/Users/uynil/projects/agents/docs/00-system/prompt-iteration-log.md)

## 文档是怎么分工的

### 面向人的入口

- [README.md](/Users/uynil/projects/agents/README.md)
  说明这个仓库是什么、有哪些内容、怎么开始

### 面向研究执行的规则

- [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
  说明研究流程、文档更新规则、经验沉淀方式

### 面向实际产出的文档

- [docs/01-projects](/Users/uynil/projects/agents/docs/01-projects)
  存放每个项目的研究笔记

- [docs/02-comparisons](/Users/uynil/projects/agents/docs/02-comparisons)
  存放多个项目之间的对比结论

### 面向方法迭代的文档

- [docs/00-system/prompt-library.md](/Users/uynil/projects/agents/docs/00-system/prompt-library.md)
  prompt 库

- [docs/00-system/evaluation-rubric.md](/Users/uynil/projects/agents/docs/00-system/evaluation-rubric.md)
  评估标准

- [docs/00-system/prompt-iteration-log.md](/Users/uynil/projects/agents/docs/00-system/prompt-iteration-log.md)
  prompt 迭代记录

## 当前状态

目前已经完成：

- 将样例项目整理到 `examples/`
- 初始化 `docs/` 文档系统
- 创建单项目目录、对比目录、方法论文档和模板
- 编写根目录 [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)

目前还没开始系统性的项目研究，因此：

- 大部分项目目录还是占位状态
- 对比文档目前主要是骨架
- 后续重点是逐个项目补研究结果

## 推荐的起步项目

如果没有特别偏好，建议先从下面两个开始：

1. [DeepSearchAgent-Demo](/Users/uynil/projects/agents/examples/DeepSearchAgent-Demo)
2. [gpt-researcher](/Users/uynil/projects/agents/examples/gpt-researcher)

原因是这两个项目都比较适合作为“建立分析框架”的起点，一个更偏阶段化工作流，一个更适合看 planner / execution 风格。

## 目标

最终希望把这个仓库沉淀成一套长期可复用的研究资产，而不只是零散笔记：

- 能持续加入新的 agent 项目
- 能稳定产出单项目研究文档
- 能逐步形成横向比较结论
- 能让 prompts 和研究方法一起迭代升级
