# 架构总览（中文）

下面是 Customized Agentic System 的高层架构图，展示主要组件与数据/交互流向。

```mermaid
flowchart LR
  subgraph 前端[Next.js 前端]
    UI[聊天页 / 模型选择 / 工具面板]
    仪表盘[运维面板 / 看板 / Wiki / 统计 / Venture]
  end

  subgraph 后端[FastAPI 后端]
    API[/api/* 路由<br/>(chat, plan, agents, stats, memory)/]
    AgentRunner[Agent 运行器 / 协调器]
    Planner[Plan / 自动排程]
    Tools[工具注册表（47 个工具，12 大类）<br/>文件 / Git / Web / 浏览器 / Shell<br/>记忆 / 知识 / 搜索 / 看板<br/>邮件 / 日历 / 文档导入 / MCP]
    LLMAdapters[LLM 适配层与 4 级降级链]
    Memory[共享 / 情景 / FIFO 记忆管理器]
    Governance[治理层 / RBAC / 速率限制 / 注入扫描]
    Audit[审计日志 / 状态迁移事件]
    Observability[统计追踪 / 链路追踪 / 端点追踪]
    MultiAgent[多 Agent 调度 / 辩论 / 分发]
    Harness[自治循环驱动<br/>Goal Queue / 检查点 / 断点续传]
    EvalGate[EvalGate<br/>LLM 评判质量门]
    Constitution[Constitution<br/>9 条不可变规则]
    Skills[Skill 注册系统<br/>可插拔能力]
    Channels[邮件机器人 / 飞书机器人]
  end

  subgraph 模型[外部与本地模型]
    DeepSeek[DeepSeek V4 API]
    Gemini[Gemini 2.5 / OpenAI GPT]
    Ollama[Ollama（本地）]
  end

  UI -->|SSE / REST| API
  仪表盘 -->|轮询 / REST| API
  API --> AgentRunner
  API --> Planner
  AgentRunner --> MultiAgent
  Planner --> AgentRunner
  AgentRunner --> LLMAdapters
  LLMAdapters --> 模型
  AgentRunner --> Tools
  Tools --> Memory
  Tools --> Governance
  Governance --> Audit
  API --> Observability
  AgentRunner --> Observability
  API --> Governance
  Governance --> Tools
  AgentRunner --> EvalGate
  AgentRunner --> Constitution
  AgentRunner --> Skills
  AgentRunner --> Harness
  Harness --> Planner
  Observability -->|日志 / jsonl| 存储[(logs/ stats/ jsonl 文件)]
  Memory -->|持久化| 存储
  Tools -->|文件 / git 操作| 存储
  MultiAgent --> AgentRunner
  Channels --> API

  classDef infra fill:#f9f,stroke:#333,stroke-width:1;
  class 模型 infra
  class 存储 infra
```

说明：
- 前端以 SSE 流为主（聊天、计划执行），并通过 REST 调用控制 API。
- 后端由 Agent、Planner、工具注册表等组成；Agent 通过 LLM 适配层调用模型，并在必要时降级与重试。
- **治理层是第一级横切关注点**：每个工具调用都要经过 RBAC、速率限制和注入扫描后才能执行。
- **Constitution (DD-48)** 定义了 9 条不可变规则，Agent 无法覆盖——包括保护治理层本身。
- **自治循环驱动 (DD-49)** 轮询 GOALS.md 中的排队任务，并从检查点自动恢复中断的计划。
- 可观测性记录每次工具调用、成本事件和状态迁移，以结构化 JSONL 审计日志存储。
- 渠道层（邮件、飞书）将系统连接到外部通讯平台。

## 计划拆解示例

系统可以将用户目标拆解为多个步骤，每个步骤分配到不同的 Agent 角色和 LLM 模型：

```mermaid
flowchart LR
  subgraph 计划[计划拆解示例]
    目标[用户目标:<br/>&#34;构建登录系统&#34;]
    目标 --> 步骤1[步骤1：调研<br/>(模型：deepseek-v4-flash)]
    目标 --> 步骤2[步骤2：实现<br/>(模型：gemini-2.5-flash)]
    目标 --> 步骤3[步骤3：测试与验证<br/>(模型：ollama/qwen2.5)]
  end

  步骤1 -->|分配给| 研究Agent[agent: research]
  步骤2 -->|分配给| 开发Agent[agent: coding]
  步骤3 -->|分配给| 测试Agent[agent: testing]

  步骤1 -->|调用| LLMAdapters
  步骤2 -->|调用| LLMAdapters
  步骤3 -->|调用| LLMAdapters

  LLMAdapters --> DeepSeek
  LLMAdapters --> Gemini
  LLMAdapters --> Ollama

  classDef highlight fill:#efe,color:#000,stroke:#333,stroke-width:1;
  class 步骤1,步骤2,步骤3 highlight
```

说明：
- 一个 `Task`（计划）可以拆成多个 `Step`，并为每个子步骤分配不同的 agent 与最合适的 LLM 模型。
- 这种按步骤选择模型的能力是项目的重要卖点，有助于在成本/速度/质量之间做精准权衡。
- LLM Mode 与 Agent 角色解耦，支持精细的成本/质量/保密性权衡。

## 关键演进（2026 年 6 月 → 2026 年 7 月）

| 领域 | 之前 | 现在 |
|------|------|------|
| 工具数 | 17 | **47** — 新增浏览器、邮件、日历、看板、知识搜索、文档导入、MCP 工具、Agent 自省诊断 |
| Agent 角色 | 6 | **27** — 原始 6 角色 + 21 个专门角色 |
| 自治理 | 无 | **CONSTITUTION.md** 9 条不可变规则 |
| 自主执行 | 无 | **自治循环驱动** 含 Goal Queue + 检查点/恢复 |
| EvalGate | 无 | **LLM 评判质量门** |
| Skills | 无 | **Skill 注册系统** |
| 渠道 | 无 | **邮件 + 飞书** 机器人集成 |
| E2E 测试 | 无 | **Playwright** 148+ specs |
| 单元测试 | ~3000 | **9823**（0 失败 / 0 flaky） |
| CI 门禁 | 基础 | **Ruff 0 / Mypy 0 / tsc 0** |
