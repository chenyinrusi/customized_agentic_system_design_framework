# Customized Agentic System Design Framework

A design philosophy and architecture framework for building **multi-agent engineering platforms** — where specialized AI agents collaborate safely, transparently, and cost-effectively on real engineering work.

## Overview

Most AI developer tools are either a single black-box assistant or a loose toolkit without project continuity. This framework proposes a different path: **governance-first, multi-agent by default, observable from day one.**

The reference implementation (Customized Agentic System) has grown substantially since this framework was first published. See the **Evolution** sections in the design documents for the latest numbers and capabilities.

## Current Reference Implementation Stats

| Metric | Value |
|--------|-------|
| Agent roles | **27** (from the original 6) |
| Built-in tools | **47 across 12 categories** (from 17) |
| Source modules | **36+** (each with MANUAL.md) |
| Unit tests | **9823** — 0 failed, 0 flaky |
| Test coverage | **80%** (~170 files at 100%) |
| CI gates | Ruff 0 · Mypy 0 · tsc 0 |
| E2E tests | Playwright — 148+ specs |
| SSE streaming P50 | 47ms |
| Design documents | 12, all implemented |

## Documents

| File | Language |
|------|----------|
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK.md` | English |
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK_CN.md` | 简体中文 |
| `architecture_en.md` | English architecture overview |
| `architecture_zh.md` | 中文架构总览 |
| `architecture_interactive.html` | Interactive architecture diagram (en/zh toggle) |

## Architecture Visualizations

- `architecture_en.md` — English architecture summary with core component relationships and plan decomposition examples.
- `architecture_zh.md` — 中文版本，包含相同结构与任务拆解示例。
- `architecture_interactive.html` — 可交互 HTML 页面，支持中英文切换、详细拆解视图和 SVG 导出。

## Videos

- [Customized Agentic System - Plan Decompose & Thanks (Playlist)](https://www.youtube.com/playlist?list=PLblsECFOUwOnwiZwgud0mXUf_diDoqRpe) — 项目介绍与计划分解演示视频，包含架构讲解与致谢。

## Core Ideas

- **Multi-agent by default** — specialized agents (Planner, Coder, Researcher, Tester, Reviewer, Memory, and 21 more) work in parallel
- **Governance-first** — every dangerous action is blocked until explicitly approved; RBAC, rate limiting, and injection scanning on every tool call
- **Observable and auditable** — every tool call, token cost, and decision is tracked in structured audit logs
- **Model routing** — LLM Mode is decoupled from agents, enabling fine-grained cost/quality trade-offs per step
- **Cross-session memory** — structured persistence of decisions, constraints, and project context (Shared / Episodic / FIFO)
- **Resilient fallback** — when one provider fails, the system routes to alternatives through a 4-level key fallback chain
- **Constitutional self-governance** — CONSTITUTION.md defines immutable invariants that no agent can override (DD-48)
- **Autonomous loop harness** — background goal queue polling, plan decomposition, and checkpoint/resume (DD-49)
- **MCP tool integration** — Model Context Protocol for extending tool capabilities at runtime

## Related Project

This framework is the design companion to the full reference implementation:
- [Customized Agentic System](https://github.com/chenyinrusi/customized_agentic_system) — the complete multi-agent engineering platform (9823 tests, 47 tools, 27 roles)

## License

MIT
