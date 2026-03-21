# Agents Research Workspace

这是一个用来研究开源 `agents` 项目的工作区。

重点不是单纯收集样例代码，而是建立一套可长期复用的学习系统，持续理解：

- agent 架构设计
- LLM orchestration 和调度模式
- prompts 的组织、编写和使用方式
- 不同项目之间的共同模式与差异
- 自己的研究方法如何持续迭代

## 仓库结构

### 代码样例

- [examples](/Users/uynil/projects/agents/examples)
  被研究的开源项目都放在这里。

### 研究文档

- [docs](/Users/uynil/projects/agents/docs)
  研究系统的主目录。

  主要分层如下：
  - [docs/system](/Users/uynil/projects/agents/docs/system)
    学习系统本身的文件：流程、prompts、评估、方法、模板
  - [docs/research](/Users/uynil/projects/agents/docs/research)
    具体 agent 项目的研究产出：单项目笔记与横向比较

### 执行规则

- [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
  这份文件负责说明研究执行规范和文档更新规则。

## 文档职责

- [README.md](/Users/uynil/projects/agents/README.md)
  给人看的入口，只说明仓库是什么、去哪里看、怎么开始。
  不记录具体 agent 项目的设计经验。

- [docs/system/README.md](/Users/uynil/projects/agents/docs/system/README.md)
  学习系统文件导航。

- [docs/research/README.md](/Users/uynil/projects/agents/docs/research/README.md)
  agent 项目研究文档导航。

- [learning-workflow.md](/Users/uynil/projects/agents/docs/system/workflow/learning-workflow.md)
  研究流程的唯一真相源。

- [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
  学习系统本身的执行规则、更新边界、经验晋升门槛、失败记录规则。
  不记录具体 agent 项目的架构或设计结论。

- [prompt-library.md](/Users/uynil/projects/agents/docs/system/prompts/prompt-library.md)
  可直接使用的研究 prompts。

- `docs/research/projects/*` 与 `docs/research/comparisons/*`
  记录具体 agent 项目的架构、调度、prompt 设计和横向比较结论。

- `docs/system/*`
  只记录学习系统本身如何运作，不记录具体 agent 项目的设计结论。

## 推荐开始方式

1. 阅读 [docs/README.md](/Users/uynil/projects/agents/docs/README.md)
2. 阅读 [docs/system/README.md](/Users/uynil/projects/agents/docs/system/README.md)
3. 阅读 [learning-workflow.md](/Users/uynil/projects/agents/docs/system/workflow/learning-workflow.md)
4. 阅读 [AGENTS.md](/Users/uynil/projects/agents/AGENTS.md)
5. 选一个项目开始
6. 用 [prompt-library.md](/Users/uynil/projects/agents/docs/system/prompts/prompt-library.md) 开始第一轮研究

## 当前状态

目前已经完成：

- 将样例项目整理到 `examples/`
- 初始化 `docs/` 文档系统
- 建立研究流程、prompt 库、模板和方法论文档
- 增加 session-close checklist、prompt smoke cases 和主题索引

目前还没有开始系统性的项目研究，因此多数项目笔记仍是占位状态。

## 推荐起步项目

如果没有特别偏好，建议先从这两个开始：

1. [DeepSearchAgent-Demo](/Users/uynil/projects/agents/examples/DeepSearchAgent-Demo)
2. [gpt-researcher](/Users/uynil/projects/agents/examples/gpt-researcher)

## 目标

希望把这个仓库逐步沉淀成一套稳定的研究资产：

- 能持续接入新的 agent 项目
- 能产出稳定的单项目笔记
- 能做横向比较
- 能让 prompts 和研究方法一起演化
