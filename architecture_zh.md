# 架构总览（中文）

下面是 Customized Agentic System 的高层架构图，展示主要组件与数据/交互流向。

```mermaid
flowchart LR
  subgraph 前端[Next.js 前端]
    UI[聊天页 / 模型选择 / 工具面板]
    仪表盘[Agent 运维面板]
  end

  subgraph 后端[FastAPI 后端]
    API[/api/* 路由\n(chat, plan, agents, stats, memory)/]
    AgentRunner[Agent 运行器 / 协调器]
    Planner[Plan / 自动驾驶（Autopilot）]
    Tools[工具注册表（17 个工具）\n文件 / Git / Web / Memory / 沙箱]
    LLMAdapters[LLM 适配层与降级链]
    Memory[SharedMemory 管理器]
    Governance[治理 / 审批模块]
    Observability[工具调用记录 / 统计 / 跟踪]
    MultiAgent[多 agent 系统]
  end

  subgraph 模型[外部与本地模型]
    DeepSeek[DeepSeek API]
    Gemini[Gemini / OpenAI]
    Ollama[Ollama (本地)]
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
  API --> Observability
  AgentRunner --> Observability
  API --> Governance
  Governance --> Tools
  Observability -->|日志 / jsonl| 存储[(logs/ stats/ jsonl 文件)]
  Memory -->|持久化| 存储
  Tools -->|文件 / git 操作| 存储
  MultiAgent --> AgentRunner

  classDef infra fill:#f9f,stroke:#333,stroke-width:1;
  class 模型 infra
  class 存储 infra
```

说明：
- 前端以 SSE 流为主（聊天、计划执行），并通过 REST 调用控制 API。
- 后端由 Agent、Planner、工具注册表等组成；Agent 通过 LLM 适配层调用模型，并在必要时降级与重试。
- Observability 与 Governance 为横切关注点：记录工具调用、成本并对危险操作做审批。

任务拆解示例（显示更细粒度的步骤与每步 LLM 模式）：

```mermaid
flowchart LR
  subgraph 计划[计划拆解示例]
    目标[用户目标:\n"构建登录系统"]
    目标 --> 步骤1[步骤1：调研\n(模型：deepseek-v4-flash)]
    目标 --> 步骤2[步骤2：实现\n(模型：gemini-2.5-flash)]
    目标 --> 步骤3[步骤3：测试与验证\n(模型：ollama/qwen2.5)]
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
