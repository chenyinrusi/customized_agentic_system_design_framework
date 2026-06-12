# Customized Agentic System Design Framework

A design philosophy and architecture framework for building **multi-agent engineering platforms** — where specialized AI agents collaborate safely, transparently, and cost-effectively on real engineering work.

## Overview

Most AI developer tools are either a single black-box assistant or a loose toolkit without project continuity. This framework proposes a different path: **governance-first, multi-agent by default, observable from day one.**

## Documents

| File | Language |
|------|----------|
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK.md` | English |
| `DESIGN_PHILOSOPHY_AND_FRAMEWORK_CN.md` | 简体中文 |
| `architecture_en.md` | English architecture overview |
| `architecture_zh.md` | 中文架构总览 |
| `architecture_interactive.html` | Interactive architecture diagram |

## Architecture Visualizations

- `architecture_en.md` — English architecture summary with core component relationships and plan decomposition examples.
- `architecture_zh.md` — 中文版本，包含相同结构与任务拆解示例。
- `architecture_interactive.html` — 可交互 HTML 页面，支持中英文切换、详细拆解视图和 SVG 导出。

## Videos

- [Customized Agentic System - Plan Decompose & Thanks (Playlist)](https://www.youtube.com/playlist?list=PLblsECFOUwOnwiZwgud0mXUf_diDoqRpe) — 项目介绍与计划分解演示视频，包含架构讲解与致谢。

## Core Ideas

- **Multi-agent by default** — specialized agents (Planner, Coder, Researcher, Tester, Reviewer, Memory) work in parallel
- **Governance-first** — every dangerous action is blocked until explicitly approved
- **Observable and auditable** — every tool call, token cost, and decision is tracked
- **Model routing** — LLM Mode is decoupled from agents, enabling fine-grained cost/quality trade-offs
- **Cross-session memory** — structured persistence of decisions, constraints, and project context
- **Resilient fallback** — when one provider fails, the system routes to alternatives

## License

MIT
