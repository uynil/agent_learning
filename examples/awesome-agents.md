repos:
https://github.com/tgoai/tgo
- `BettaFish`
- `DeepSearchAgent-Demo`
- `MiroFish`
- `autoresearch`
- `chatwoot`
- `everything-claude-code`
- `gpt-researcher`
- https://github.com/rangeking/self-evolving-agent
- git@github.com:shakil1819/Multi-Agent-Data-Analyst.git
- https://github.com/VaishnaviSh14/MultiAgent-Data-Analyst



1. Anomstack ⭐ 数据指标审计平台
GitHub: andrewm4894/anomstack 
开源时间: 2025-07 发布，持续活跃
核心能力:
数据审计: 自动监控多数据源（BigQuery/Snowflake/ClickHouse/DuckDB/Redshift/SQLite）的指标质量
异常挖掘: 基于 PyOD 库的多种异常检测算法（Isolation Forest、LSTM、AutoEncoder 等）
LLM Agent 审计: 内置 anomaly-agent，用 LLM 判断指标是否异常并生成审计解释 
反馈闭环: 支持人工反馈（👍/👎）优化检测准确性
适用场景: 金融指标监控、系统性能审计、业务数据质量检查


2. Argos ⭐ 微软开源可解释审计规则生成
GitHub: microsoft/argos 
开源时间: 2025-04（微软官方）
核心能力:
智能规则挖掘: 用 LLM（GPT-4）自动生成时间序列异常检测规则，无需人工编写
可解释审计: 生成的规则具有人类可读的解释（优于黑盒深度学习）
性能提升: 比传统深度学习方法（AnomalyTransformer、LSTMAD）F1 分数提升 28.3% 
规则溯源: 所有异常判定都可追溯到生成的规则逻辑
适用场景: 需要合规审计和可解释性要求的场景（金融监管、工业物联网）
关键特性:
自动生成可解释的异常检测规则（替代人工阈值设置）

3. anomaly-agent ⭐ 专门的时间序列审计 Agent
GitHub: andrewm4894/anomaly-agent 
开源时间: 2025-02（Anomstack 作者新项目）
核心能力:
Agentic 异常检测: 基于 LangGraph 状态机的多步骤审计流程
验证链: 检测→验证→解释，确保审计结果准确
Pydantic 验证: 类型安全的数据模型，防止脏数据污染审计结果 
轻量级: 单 API 调用 AnomalyAgent().detect_anomalies(df)
技术栈: LangChain + LangGraph + OpenAI，支持流式输出和异步处理

4. Anomalib ⭐ 工业视觉异常挖掘
GitHub: open-edge-platform/anomalib 
开源时间: 持续更新至 2026-03（Intel 官方维护）
核心能力:
视觉模式挖掘: 在图像/视频中自动检测和定位异常区域（缺陷检测）
少样本学习: 仅需正常样本即可训练（无需标注异常数据）
模型丰富: PatchCore、STFPM、AnomalyDINO（基于 DINOv2）等 15+ SOTA 算法 
边缘部署: 支持导出 OpenVINO 格式，可在工业边缘设备运行
适用场景: 工业质检（电路板、纺织品、汽车零件缺陷审计）

5. Multi-Agent-Data-Analyst 数据血缘审计
GitHub: shakil1819/MultiAgent-DataAnalyst 
开源时间: 2025-03
审计特性:
数据溯源: Agent Orchestrator 追踪数据处理全流程
质量门禁: Data Quality Agent 检查缺失值、异常值、格式一致性

### resources
https://github.com/EvoAgentX/Awesome-Self-Evolving-Agents