# Architecture Overview (English)

Below is a high-level architecture diagram of the Customized Agentic System. It highlights major components and the main data/interaction flows.

```mermaid
flowchart LR
  subgraph Frontend[Next.js Frontend]
    UI[Chat Page / Model Selector / Toolbox]
    AgentWorkspace[Agent Dashboard / Kanban / Wiki / Stats / Venture]
  end

  subgraph Backend[FastAPI Backend]
    API[/api/* endpoints<br/>(chat, plan, agents, stats, memory)/]
    AgentRunner[Agent Runner / Orchestrator]
    Planner[Plan / Autopilot]
    Tools[Tool Registry (47 tools, 12 categories)<br/>File / Git / Web / Browser / Shell<br/>Memory / Knowledge / Search / Kanban<br/>Email / Calendar / Ingestion / MCP]
    LLMAdapters[LLM Adapters & 4-level Fallback Chain]
    Memory[SharedMemory / Episodic / FIFO Manager]
    Governance[Governor / RBAC / Rate Limit / Injection Scan]
    Audit[Audit Log / StateTransitionEvent]
    Observability[StatsTracker / Tracer / EndpointTracker]
    MultiAgent[MultiAgentSystem / Debate / Dispatch]
    Harness[Autonomous Loop Harness<br/>Goal Queue / Checkpoint / Resume]
    EvalGate[EvalGate<br/>LLM-as-Judge quality gate]
    Constitution[Constitution<br/>9 immutable invariants]
    Skills[Skill Registry<br/>Pluggable capabilities]
    Channels[Email Bot / Feishu Bot]
  end

  subgraph Providers[External & Local Models]
    DeepSeek[DeepSeek V4 API]
    Gemini[Gemini 2.5 / OpenAI GPT]
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
  Observability -->|logs/jsonl| Storage[(logs/ stats/ jsonl files)]
  Memory -->|persist| Storage
  Tools -->|file/git ops| Storage
  MultiAgent --> AgentRunner
  Channels --> API

  classDef infra fill:#f9f,stroke:#333,stroke-width:1;
  class Providers infra
  class Storage infra
```

Notes:
- Frontend communicates with Backend primarily via SSE for streaming chat and plan execution, plus REST for control APIs.
- Backend composes agents, planner and tool registry; agents call LLM adapters which route to providers with 4-level fallback and retries.
- **Governance is a first-class cross-cutting concern**: every tool call passes through RBAC, rate limiting, and injection scanning before execution.
- **Constitution (DD-48)** defines 9 immutable invariants that no agent can override — including protection of the governance layer itself.
- **Autonomous Loop Harness (DD-49)** polls GOALS.md for queued work and auto-resumes interrupted plans from checkpoints.
- Observability records every tool call, cost event, and state transition in structured JSONL audit logs.
- Channels (email, Feishu) connect the system to external communication platforms.

## Plan Decomposition Example

The system can decompose a user goal into multiple steps, each assigned to a different agent role and LLM mode:

```mermaid
flowchart LR
  subgraph Plan[Plan Decomposition]
    Goal[User Goal:<br/>&#34;Build login system&#34;]
    Goal --> Step1[Step 1: Research<br/>(model: deepseek-v4-flash)]
    Goal --> Step2[Step 2: Implement<br/>(model: gemini-2.5-flash)]
    Goal --> Step3[Step 3: Test & Verify<br/>(model: ollama/qwen2.5)]
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
- The LLM Mode is decoupled from the agent role, enabling fine-grained cost/quality/confidentiality trade-offs.

## Key Evolution (June 2026 → July 2026)

| Area | Before | Now |
|------|--------|-----|
| Tools | 17 | **47** — added browser, email, calendar, kanban, knowledge search, document ingestion, MCP tools, agent introspection |
| Agent roles | 6 | **27** — original 6 + 21 specialized roles |
| Self-governance | None | **CONSTITUTION.md** with 9 invariants |
| Autonomous execution | None | **Loop Harness** with goal queue + checkpoint/resume |
| EvalGate | None | **LLM-as-Judge** quality gate |
| Skills | None | **Skill Registry** for pluggable capabilities |
| Channels | None | **Email + Feishu** bot integrations |
| E2E testing | None | **Playwright** 148+ specs |
| Unit tests | ~3000 | **9823** (0 failed, 0 flaky) |
| CI gates | Basic | **Ruff 0 / Mypy 0 / tsc 0** |
