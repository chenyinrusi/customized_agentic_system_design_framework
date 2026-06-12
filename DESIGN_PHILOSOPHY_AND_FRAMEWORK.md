# Design Philosophy and Architecture Framework

## 1. Design Philosophy

### 1.1 Clear Purpose: "AI as a Trustworthy System"
The core idea is not to make one smarter AI. It is to build a system where multiple specialized AI agents can work together safely, transparently, and reliably.

- **Trust over novelty**: the product must be predictable and auditable, not just impressive.
- **Human control first**: every dangerous action is gated by governance, approval, and human consent.
- **Transparency by design**: every decision, tool call, and cost event should be observable.
- **Resilience with fallback**: the system must continue working even when an LLM provider fails.
- **Memory and continuity**: the system should remember project decisions across sessions, not just within a single chat.

### 1.2 What makes this idea different

- Many AI systems are built as a single monolithic assistant.
- This project is intentionally modular: each agent has a focused role, and the system orchestrates them.
- The goal is to move from "AI assistant" to "AI-enabled engineering platform".
- The product is not just about generating outputs — it is about managing risk, cost, and long-lived engineering work.
- The system intentionally decouples an agent's brain from the rest of the workflow: the LLM Mode is treated as the brain, while agents orchestrate tasks and execute tools.

This separation means the brain can be swapped at any point in a project or any step of an agent's work, enabling fine-grained mode selection for confidentiality, cost, and quality.

## 2. Architecture Framework

### 2.1 Core layers

The architecture is organized into a small number of clearly separated layers:

1. **Orchestration layer**
   - Controls the lifecycle of tasks.
   - Manages multiple agents and their dependencies.
   - Routes work to the right role at the right time.

2. **Reasoning layer**
   - Produces plans and task decompositions.
   - Uses stronger models for strategy and review.
   - Supports live plan revision based on feedback.

3. **Execution layer**
   - Runs code generation, testing, review, and knowledge capture.
   - Executes tools such as shell commands, file writes, git operations, and browser actions.

4. **Governance layer**
   - Enforces RBAC and approval policies.
   - Captures audit events and risk metadata.
   - Controls dangerous operations and budget-sensitive decisions.

5. **Observability layer**
   - Logs all tool calls and agent status.
   - Tracks cost, token usage, and execution chains.
   - Makes the system visible to users.

6. **Memory layer**
   - Persists project context and decisions.
   - Provides structured cross-session memory.
   - Compresses and prunes data to keep history useful.

### 2.2 Agent roles

The system is built around specialized agents instead of a single general-purpose assistant:

- **Planner**: decomposes goals into steps, selects models, and assigns work.
- **Coding**: writes implementation code and production artifacts.
- **Research**: looks up patterns, libraries, and architectural options.
- **Testing**: writes and executes validation tests.
- **Review**: audits code quality, security, and best practices.
- **Memory**: captures decisions, constraints, and historical context.

Each role is designed to have a separate responsibility and can run in parallel with others.

A key principle is that LLM Mode is a smaller-grained decision than an agent. That means a complex task step can use a higher-end mode while simpler steps use a cheaper mode, and a single project can route different data or steps to different modes to avoid exposing the whole idea to one provider.

### 2.3 Multi-agent orchestration model

- The system uses a task queue and a scheduler to execute multiple agents concurrently.
- Dependencies are resolved so work can proceed in parallel when safe.
- Agents can exchange findings through a lightweight message bus and shared memory.
- A synthesis step aggregates results and ensures consistency.

### 2.4 Governance and safety principles

- **Prevent before permit**: risky operations are blocked until explicitly approved.
- **Role-aware permissions**: different agent roles and user roles have different capabilities.
- **Budget awareness**: model selection is influenced by cost budgets and usage thresholds.
- **Audit-first**: every operation is recorded with context, outcome, and approval state.

### 2.5 Observability and diagnosis

- The system is designed for live monitoring of running agents.
- Status dashboards display agent progress, token use, costs, and pending approvals.
- Execution traces link user requests to specific tool calls and agent decisions.
- This transparency is a deliberate product feature, not an afterthought.

### 2.6 Memory and continuity

- Project memory stores decisions, constraints, architecture notes, and risks.
- Memory is structured rather than raw chat history.
- The system automatically injects relevant memory into future agent prompts.
- Aging, deduplication, and compression keep memory useful and small.

## 3. High-level product narrative

### 3.1 The problem being solved

Most AI developer tools today either:
- act as a single black-box assistant; or
- provide a loose toolkit without long-term project continuity.

This product solves the gap between short-lived AI interactions and real engineering work by offering:

- parallel execution across multiple specialized AI roles
- strict operational safety and approval controls
- transparent cost and behavior tracking
- persistent memory across development sessions

### 3.2 The target value proposition

- Faster software delivery through parallel agent work.
- Safer automation through governance controls.
- Lower risk through observability and audit trails.
- Better continuity through cross-session memory.
- Practical cost management through model routing and fallback.

### 3.3 Why this matters

Engineering work is inherently collaborative, iterative, and stateful.
A project that loses context or runs unchecked is not useful in production.
This design is meant to bring AI into engineering as a controlled, measurable partner rather than a wild experiment.

## 4. Distinctive design choices

### 4.1 Multi-agent by default

Most systems treat multiple agents as optional or experimental.
This design makes them the default way to solve nontrivial engineering problems.

### 4.2 Governance as a first-class feature

Instead of adding governance later, this design assumes every action may require approval.
That makes the product suitable for practical teams and enterprise usage.

### 4.3 Memory as a product feature

The system does not simply keep conversation history.
It preserves decisions, constraints, and strategic context that matter across days.

### 4.4 Fallbacks over failure

Rather than fail when one API is down, the system falls back through available models and providers.
This creates a more resilient and reliable user experience.

### 4.5 Observability for trust

The product is built with the assumption that users should always know what happened and why.
That is the core difference between an experimental tool and a trusted engineering platform.

## 5. How to describe it without code

If you want to express this idea without sharing implementation details, frame it as:

- "A multi-agent engineering platform" rather than "a Python project".
- "A governance-first AI system" rather than "a set of libraries".
- "A project memory engine" rather than "a source-controlled database".
- "A cost-aware model router" rather than "provider fallback tables".

The value is in the design pattern, not the code itself.

---

## Appendix: Key design statements

- "This product treats AI as a set of cooperative specialists, not a single oracle."
- "It is built to be observable, auditable, and budget-aware from day one."
- "It remembers project decisions, instead of forcing users to re-explain context."
- "It fails gracefully by falling back to alternative models and providers."
- "It prevents dangerous operations by default, and only acts when the user approves."

---

## Acknowledgements

Thanks to Hunter Baun for creating Deepseek TUI. It removed the brakes on my development process and let me build this product quickly without worrying too much about cost. 

Thanks to Andrej Karpathy for the LLM Wiki – my memory system was inspired by that idea, and I am still studying his autoresearch work.

Thanks to DeepSeek as well. Because of its affordable pricing, I was able to build this project in the first place. I believe people in our era should thank DeepSeek’s founder, Liang Wenfeng. He has preserved access to AI for ordinary users, and without him, there would be no AI equity for the public.

Finally, I actually do not want to thank Claude. It is expensive and has a frustrating 5-hour usage cap. But sometimes when no other model can solve a bug, Claude can. My suggestion is to use Claude for design and hard problems, while using DeepSeek for regular development. After all, buying an 80-point product for $1 or a 90-point product for $100， cost over performance is a much better choice for cost-sensitive consumers, and DeepSeek is likely the better option for them.

A video walkthrough of this project is available on YouTube: [Customized Agentic System - Plan Decompose & Thanks](https://www.youtube.com/playlist?list=PLblsECFOUwOnwiZwgud0mXUf_diDoqRpe).
