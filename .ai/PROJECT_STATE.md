# Project State

## 1. 项目概况
- **项目名称**：DSH Multi-Model Multi-Agent Collaborative Development Environment
- **当前状态**：初始化完成，多 Agent 协作环境与持久化系统已就绪
- **基础环境**：DeepSeek Harness (0.1.1-rc.2) + CLIProxyAPI (CPA Proxy)
- **默认调度模式**：多模型多 Agent 协作模式 (`multi-agent` preset)

---

## 2. 协作角色矩阵与当前会话绑定

| 角色 | 调度工具 | CPA 实际模型 | 推理深度 | 当前子会话 ID (Continuable) | 状态 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Orchestrator** | Top-level Session | `sub-all/gpt-5.6-sol` | **Max** | *(Top-level session)* | Active |
| **GPT Core / Architect** | `subagent_gpt_core` | `sub-all/gpt-5.6-luna` | **Max** | - | Ready |
| **Gemini Frontend** | `subagent_gemini_frontend` | `sub-all/gemini-3.7-flash-high` | 标准 | - | Ready |
| **Implementer** | `subagent_implementer` | `sub-all/gemini-3.7-flash-high` | 标准 | - | Ready |
| **Reviewer** | `subagent_reviewer` | `sub-all/claude-opus-4-6-thinking` | **xhigh** | - | Ready |

---

## 3. 当前里程碑 (Current Milestone)
- **Milestone 1**: 多模型多 Agent 环境搭建与原生 Continuable Subagent 链路打通 **[COMPLETED]**
- **Milestone 2**: 共享持久化状态机制 (`.ai/` + `AGENTS.md`) 建立与规范化 **[COMPLETED]**
- **Milestone 3**: 实际业务与工程任务多 Agent 协作接续 **[IN PROGRESS]**

---

## 4. 关键架构事实
- **模型路由**：统一由 CPA (`sub-all`) 管理，无需本地手动维护不同厂商的 API Key。
- **状态流转**：Git 仓库代码与 `.ai/` 文档为权威状态，跨会话不依赖聊天历史直接传参。
- **权限安全**：Reviewer 默认禁止写操作；前端与后端遵循目录责任隔离。
