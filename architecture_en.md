# Architecture Overview (English)

Below is a high-level architecture diagram of the Customized Agentic System. It highlights major components and the main data/interaction flows.

```mermaid
flowchart LR
  subgraph Frontend[Next.js Frontend]
    UI[Chat Page / Model Selector / Toolbox]
    AgentWorkspace[Agent Dashboard]
  end

  subgraph Backend[FastAPI Backend]
    API[/api/* endpoints\n(chat, plan, agents, stats, memory)/]
    AgentRunner[Agent Runner / Orchestrator]
    Planner[Plan / Autopilot]
    Tools[Tool Registry (17 tools)\nFile / Git / Web / Memory / Sandbox]
    LLMAdapters[LLM Adapters & Fallback]
    Memory[SharedMemoryManager]
    Governance[Governor / Approval]
    Observability[ToolCallLogger / StatsTracker / Tracer]
    MultiAgent[MultiAgentSystem]
  end

  subgraph Providers[External & Local Models]
    DeepSeek[DeepSeek API]
    Gemini[Gemini / OpenAI]
    Ollama[Ollama (local)]
  end

  UI -->|SSE / REST| API
  AgentWorkspace -->|polling / REST| API
  API --> AgentRunner
  API --> Planner
  AgentRunner --> MultiAgent
  Planner --> AgentRunner
  AgentRunner --> LLMAdapters
  LLMAdapters --> Providers
  AgentRunner --> Tools
  Tools --> Memory
  API --> Observability
  AgentRunner --> Observability
  API --> Governance
  Governance --> Tools
  Observability -->|logs/jsonl| Storage[(logs/ stats/ jsonl files)]
  Memory -->|persist| Storage
  Tools -->|file/git ops| Storage
  MultiAgent --> AgentRunner

  classDef infra fill:#f9f,stroke:#333,stroke-width:1;
  class Providers infra
  class Storage infra
```

Notes:
- Frontend communicates with Backend primarily via SSE for streaming chat and plan execution, plus REST for control APIs.
- Backend composes agents, planner and tool registry; agents call LLM adapters which route to providers with fallback and retries.
- Observability and Governance are cross-cutting: they record tool calls, costs and gate dangerous operations.

Plan decomposition example (finer-grained steps and per-step LLM modes):

```mermaid
flowchart LR
  subgraph Plan[Plan Decomposition]
    Goal[User Goal:\n"Build login system"]
    Goal --> Step1[Step 1: Research\n(model: deepseek-v4-flash)]
    Goal --> Step2[Step 2: Implement\n(model: gemini-2.5-flash)]
    Goal --> Step3[Step 3: Test & Verify\n(model: ollama/qwen2.5)]
  end

  Step1 -->|assigned to| AgentResearch[agent: research]
  Step2 -->|assigned to| AgentCoding[agent: coding]
  Step3 -->|assigned to| AgentTesting[agent: testing]

  Step1 -->|calls| LLMAdapters
  Step2 -->|calls| LLMAdapters
  Step3 -->|calls| LLMAdapters

  LLMAdapters --> DeepSeek
  LLMAdapters --> Gemini
  LLMAdapters --> Ollama

  classDef highlight fill:#efe,color:#000,stroke:#333,stroke-width:1;
  class Step1,Step2,Step3 highlight
```

Notes:
- A single `Task` (Plan) can be decomposed into multiple `Steps` with different agents and different LLM models for each step.
- This enables assigning the best-suited model per subtask (research vs. code generation vs. verification), which is a core capability of this project.
