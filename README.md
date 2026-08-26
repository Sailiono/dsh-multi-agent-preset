# DSH Multi-Model Multi-Agent Collaborative Development Preset

[![DeepSeek Harness](https://img.shields.io/badge/DSH-v0.1.1--rc.2-blue)](https://github.com/deepseek-ai/deepseek-harness)
[![Architecture](https://img.shields.io/badge/Architecture-Continuable_Subagents-green)]()
[![License](https://img.shields.io/badge/License-MIT-purple)]()

基于 **DeepSeek Harness (DSH)** 与 **CLIProxyAPI (CPA)** 模型池的多模型、多 Agent、可持续接续协同开发预设（Agent Preset）。

---

## 🌟 核心理念与架构

```
                             ┌───────────────────────────────┐
                             │       User Requirement        │
                             └──────────────┬────────────────┘
                                            │
                                            ▼
                             ┌───────────────────────────────┐
                             │         Orchestrator          │
                             │   (CPA: sub-all/gpt-5.6-sol)  │
                             │      推理深度: Max            │
                             │  Task Decomposition & Router  │
                             └──────┬───┬───────┬────┬───────┘
                                    │   │       │    │
            ┌───────────────────────┘   │       │    └───────────────────────┐
            ▼                           ▼       ▼                            ▼
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│   GPT Core / Arch     │   │    Gemini Frontend    │   │      Implementer      │   │       Reviewer        │
│ (sub-all/gpt-5.6-luna)│   │(gemini-3.7-flash-high)│   │(gemini-3.7-flash-high)│   │(claude-opus-4-6-think)│
│    推理深度: Max      │   │                       │   │                       │   │    推理深度: xhigh    │
│  Architecture & Logic │   │  UI / UX / Frontend   │   │ Concrete Code & Test  │   │ Independent Code Audit│
└───────────────────────┘   └───────────────────────┘   └───────────────────────┘   └───────────────────────┘
            │                           │                           │                           │
            └───────────────────────────┴─────────────┬─────────────┴───────────────────────────┘
                                                      │
                                                      ▼
                                       ┌─────────────────────────────┐
                                       │   Shared Repository Memory  │
                                       │   AGENTS.md & .ai/ State    │
                                       └─────────────────────────────┘
```

- **高阶推理模型负责顶层与核心**：GPT-5.6 Sol (Max) 负责总控调度；GPT-5.6 Luna (Max) 负责系统架构与复杂决策。
- **高性价比高表现模型负责具体实现**：Gemini 3.7 Flash 负责前端视觉实现、UI 组件开发以及核心业务编码与测试。
- **独立模型负责只读代码审计**：Claude Opus 4.6 Thinking (xhigh) 负责独立 Review，权限上通过工具过滤器严格锁定为只读。
- **全局统一网关**：底层模型全部通过 CLIProxyAPI (`sub-all`) 调用，告别人工频繁切换多平台账号。
- **原生持续会话 (Continuable)**：子 Agent 保持独立的持久化 Session，支持 `send_message` 无缝接续上下文。
- **Git 仓库作为 Source of Truth**：会话仅作为临时通道，所有关键决策、任务状态与 handoff 均沉淀在仓库中。

---

## 👥 角色分工与模型绑定

| Agent 角色 | 调度入口 / 工具 | 绑定模型 (`provider/modelId`) | 推理深度 | 核心职责 | 权限策略 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Orchestrator** | 顶层会话 / 主 Agent | `sub-all/gpt-5.6-sol` | **Max** | 需求理解、任务拆解、Subagent 调度、并行控制、冲突仲裁、验收集成 | 全工具（Shell、文件读写、子代理调度、任务管理） |
| **GPT Core / Architect** | `subagent_gpt_core` | `sub-all/gpt-5.6-luna` | **Max** | 软件架构设计、核心接口与数据模型、复杂算法、疑难 Bug 根因分析、关键重构 | 全工具，专注系统设计与核心代码 |
| **Gemini Frontend** | `subagent_gemini_frontend` | `sub-all/gemini-3.7-flash-high` | 标准 | Web 前端、UI/UX 设计、React/Vue/CSS/Tailwind、响应式布局、动效与视觉一致性 | 全工具，目录所有权限定于前端相关目录 |
| **Implementer** | `subagent_implementer` | `sub-all/gemini-3.7-flash-high` | 标准 | 冻结设计后的代码实现、业务功能、CRUD、工具脚本、Bug 修复、单元/集成测试 | 全工具，专注具体代码编写与验证 |
| **Reviewer** | `subagent_reviewer` | `sub-all/claude-opus-4-6-thinking` | **xhigh** | 独立代码审查、逻辑 Bug、边界条件、并发安全、安全性审查、测试覆盖率检查 | **只读权限**（禁止直接调用 `write` / `edit`，仅输出审查 findings） |

---

## 📁 目录结构与状态机制

```text
├── AGENTS.md                   # 全局协同规范与 Agent 行为守则
├── .ai/
│   ├── PROJECT_STATE.md        # 项目全局状态、架构概览、当前 Milestone 进度
│   ├── TASKS.md                # 结构化任务看板 (TODO / IN_PROGRESS / REVIEW / DONE)
│   ├── DECISIONS.md            # 架构与技术决策记录 (ADR)
│   └── handoffs/               # 结构化交接记录目录
│       ├── README.md           # 交接单使用说明
│       └── 0001-template.md    # 标准任务交接单模板
├── .dsh/
│   └── presets/
│       └── multi-agent/        # DSH Preset 组装定义
│           ├── agent.cordis.yml
│           └── preset.yml
└── README.md                   # 项目说明
```

---

## 🚀 快速开始与安装

### 1. 安装预设到本地 DSH
将本预设复制到 DSH 的用户 Preset 目录：

```bash
mkdir -p ~/.dsh/.agent-presets/multi-agent
cp -r .dsh/presets/multi-agent/* ~/.dsh/.agent-presets/multi-agent/
```

### 2. 设置默认模型与预设
在 `~/.dsh/settings.yaml` 中配置：

```yaml
agent-default-model:
  model: gpt-5.6-sol
  provider: sub-all
  reasoningEffort: max

agent-presets:
  default: multi-agent
```

### 3. 启动 DSH Web 界面
```bash
dsh web
```

---

## 💬 常用调度自然语言指令

在会话中向 Orchestrator 发送指令即可自动调度：

> **“继续开发当前项目。让 GPT Core 先检查架构和当前状态，Gemini Frontend 继续前端工作，Implementer 完成已经冻结的后端任务，最后让 Reviewer 独立审核。无依赖任务可以并行。”**

---

## 📄 License
[MIT License](LICENSE)
