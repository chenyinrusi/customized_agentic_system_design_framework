# Customized Agentic System Design Framework

A design philosophy and architecture framework for building **multi-agent engineering platforms** — where specialized AI agents collaborate safely, transparently, and cost-effectively on real engineering work.

## Overview

Most AI developer tools are either a single black-box assistant or a loose toolkit without project continuity. This framework proposes a different path: **governance-first, multi-agent by default, observable from day one.**

## Documents

| File | Language |
|------|----------|
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK.md` | English |
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK_CN.md` | 简体中文 |

## Core Ideas

- **Multi-agent by default** — specialized agents (Planner, Coder, Researcher, Tester, Reviewer, Memory) work in parallel
- **Governance-first** — every dangerous action is blocked until explicitly approved
- **Observable and auditable** — every tool call, token cost, and decision is tracked
- **Model routing** — LLM Mode is decoupled from agents, enabling fine-grained cost/quality trade-offs
- **Cross-session memory** — structured persistence of decisions, constraints, and project context
- **Resilient fallback** — when one provider fails, the system routes to alternatives

## License

MIT
