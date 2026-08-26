# Multi-Model & Multi-Agent Collaboration Specification (DSH + CPA)

本项目采用 **DeepSeek Harness (DSH)** 结合 **CLIProxyAPI (CPA)** 模型池架构，构建了一套多模型、多角色、可持续接续的协同开发环境。

---

## 1. 核心架构与角色分工

```
                             ┌───────────────────────────────┐
                             │       User Requirement        │
                             └──────────────┬────────────────┘
                                            │
                                            ▼
                             ┌───────────────────────────────┐
                             │         Orchestrator          │
                             │   (CPA: sub-all/gpt-5.6-luna) │
                             │  Task Decomposition & Router  │
                             └──────┬───┬───────┬────┬───────┘
                                    │   │       │    │
            ┌───────────────────────┘   │       │    └───────────────────────┐
            ▼                           ▼       ▼                            ▼
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│   GPT Core / Arch     │   │    Gemini Frontend    │   │      Implementer      │   │       Reviewer        │
│(sub-all/gpt-5.6-luna) │   │(gemini-3.7-flash-high)│   │(sub-all/deepseek-v4-p)│   │(claude-opus-4-6-think)│
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

### 角色配置与模型绑定矩阵

| Agent 角色 | 核心职责 | 调度工具名称 | CPA 绑定模型 (`provider/modelId`) | 工具权限与沙箱策略 |
| :--- | :--- | :--- | :--- | :--- |
| **Orchestrator** | 需求理解、任务拆解、Subagent 调度、并行控制、冲突消解、最终集成与测试 | 主 Agent / 顶层会话 | `sub-all/gpt-5.6-luna` | 全工具访问（Shell、文件读写、子代理调度、任务管理） |
| **GPT Core / Architect** | 软件架构设计、核心接口与数据模型、复杂算法、疑难 Bug 根因分析、关键重构 | `subagent_gpt_core` | `sub-all/gpt-5.6-luna` | 全工具访问，专注系统设计与核心代码 |
| **Gemini Frontend** | Web 前端、UI/UX 设计、React/Vue/CSS/Tailwind、响应式布局、动效与视觉一致性 | `subagent_gemini_frontend` | `sub-all/gemini-3.7-flash-high` | 全工具访问，目录所有权限定于前端相关目录 |
| **Implementer** | 冻结设计后的代码实现、业务功能、CRUD、工具脚本、Bug 修复、单元/集成测试 | `subagent_implementer` | `sub-all/deepseek-v4-pro` | 全工具访问，专注具体代码编写与验证 |
| **Reviewer** | 独立代码审查、逻辑 Bug、边界条件、并发安全、安全性审查、测试覆盖率检查 | `subagent_reviewer` | `sub-all/claude-opus-4-6-thinking` | **只读权限**（禁止直接调用 `write` / `edit`，仅输出审查 findings） |

---

## 2. 状态共享与记忆机制 (Source of Truth)

为了避免跨 Agent 超长上下文复制与上下文漂移，**Git 仓库文件永远作为唯一的 Source of Truth**。会话上下文仅作为辅助执行通道。

### 共享目录规范

```text
.
├── AGENTS.md                   # 本协同规范（全局行为准则）
├── .ai/
│   ├── PROJECT_STATE.md        # 项目全局状态、架构概览、当前 Milestone 进度
│   ├── TASKS.md                # 任务看板（Backlog / In Progress / Review / Done）
│   ├── DECISIONS.md            # 架构与技术决策记录 (ADR)
│   └── handoffs/               # Agent 之间的结构化交接记录
│       ├── 0001-template.md    # 交接模板
│       └── ...                 # 任务交接具体文件 (如: 2026-08-26-core-api-design.md)
```

### Agent 开工前检查清单 (Pre-flight Checklist)
每个 Agent 被唤醒或开始任务前，**必须按顺序执行以下动作**：
1. 读取 `AGENTS.md` 了解协同规则与自身职责边界。
2. 读取 `.ai/PROJECT_STATE.md` 与 `.ai/TASKS.md` 确认当前项目状态与所领取的任务编号。
3. 读取 `.ai/DECISIONS.md` 确认已冻结的架构设计和规范约束。
4. 如有上游交接文档，阅读 `.ai/handoffs/` 中对应的最新交接单。
5. 检查与自身任务相关的实际代码文件。

### Agent 完工后更新清单 (Post-flight Checklist)
1. 运行相关测试（如有），确保没有破坏现有功能。
2. 在 `.ai/TASKS.md` 中更新任务状态。
3. 若产出需要交接给下一位 Agent（如 Architect 产出接口设计交给 Implementer 或 Frontend），在 `.ai/handoffs/` 写入交接单。
4. Orchestrator 在阶段完成时汇总更新 `.ai/PROJECT_STATE.md`。

---

## 3. 会话持续性 (Session Persistence & Resume)

所有专用子 Agent 均配置为 `backgroundMode: continuable`：
1. **首次启动**：Orchestrator 调用 `subagent_xxx` 工具，运行时会立即返回一个持久化的 `subagentId`（例如 `started subagent <childId>`），子 Agent 在后台启动并建立持久化会话记录。
2. **后续追加 / 继续会话**：Orchestrator 调用 `send_message(subagent_id: "<childId>", message: "...")` 向该 Agent 追加后续指令，子 Agent 会在已有上下文基础上继续工作，无需重新传达历史背景。
3. **查看活动状态**：Orchestrator 可随时调用 `list_agents()` 查看所有存活/已保存的子 Agent ID 与状态 (`running` / `idle` / `ready`)。
4. **冷恢复 (Cold Resume)**：当 DSH 重新启动或会话从磁盘恢复时，DSH 的 continuable 机制会自动从 `~/.dsh/sessions/` 重建子 Agent 描述符（包括指定的模型、persona 与工具过滤器），确保上下文和身份一致。

---

## 4. 并行调度与隔离规则

1. **并行执行**：Orchestrator 识别出无依赖关系的独立任务时（例如：Frontend 开发前端界面与 Implementer 编写后端数据解析脚本），在同一次回复中同时发起多个 `subagent_xxx` 调用。
2. **目录 Ownership 隔离**：
   - `Gemini Frontend`：主要负责前端目录（如 `web/`, `frontend/`, `ui/`, `components/`, `styles/` 等）。
   - `Implementer`：负责核心逻辑实现与测试目录（如 `src/`, `lib/`, `tests/`, `scripts/` 等）。
   - `GPT Core / Architect`：负责架构设计、通用接口定义、根配置文件。
   - `Reviewer`：全项目只读，审查 Git Diff 与提交内容。
3. **冲突解决**：若发生文件修改冲突或接口不一致，统一由 Orchestrator 介入协调，必要时交由 `GPT Core / Architect` 进行仲裁。

---

## 5. 交互与指令示例

### 自然语言调度指令示例
> **用户指令**：“继续开发当前项目。让 GPT Core 先检查架构和当前状态，Gemini Frontend 继续前端工作，Implementer 完成已经冻结的后端任务，最后让 Reviewer 独立审核。无依赖任务可以并行。”

### Orchestrator 执行链路
1. Orchestrator 读取 `.ai/PROJECT_STATE.md` 和 `.ai/TASKS.md`。
2. Orchestrator 调用 `subagent_gpt_core(prompt="检查当前架构与.ai/DECISIONS.md，输出核心接口规范")`。
3. Architect 结算后，Orchestrator 在同一轮次并发调用：
   - `subagent_gemini_frontend(prompt="根据Architect接口规范实现前端页面UI组件")`
   - `subagent_implementer(prompt="根据Architect接口规范实现后端数据流与测试用例")`
4. 两者完成并提交 handoff 后，Orchestrator 调用：
   - `subagent_reviewer(prompt="请对本次修改进行独立Code Review，检查安全与边界问题，输出评审意见")`
5. Orchestrator 运行测试，根据 Reviewer 反馈决定是否微调，最终更新 `.ai/PROJECT_STATE.md` 并向用户汇报。
