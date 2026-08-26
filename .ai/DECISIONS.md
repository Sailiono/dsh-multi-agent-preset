# Architecture Decisions (ADR)

## ADR-001: 采用 DSH 原生 Continuable Subagent 架构结合 CPA 模型池

### 上下文与背景
需要构建一个多模型、多 Agent 协作开发环境：
- GPT-5.6 / Codex 负责系统架构、接口设计与决策
- Gemini 3.7 负责前端、UI 与视觉实现
- DeepSeek V4 负责常规功能实现与自动化测试
- Claude Opus 4.6 负责独立代码审查
- 所有模型通过局域网内 CLIProxyAPI (CPA) 代理统一调度。

### 决策内容
1. **采用 DSH 原生 `dsh-tool-subagent` + `dsh-agent-presets` 机制**：
   - 经源码审计，DSH 0.1.1-rc.2 已具备完整的 `prepareContinuable`、`agentOptions`（独立 provider/model 绑定）、`persona` 注入、`toolFilter` 权限隔离及 `send_message` 会话接续能力。
   - 原生架构直接运行在 Cordis 微内核之上，无需引入 `pi2dsh` 或第三方适配层，避免多层包装带来的上下文丢失与性能损耗。
2. **通过 CPA 代理 (`sub-all`) 分流不同模型与推理深度**：
   - Orchestrator: `sub-all/gpt-5.6-sol` (推理深度: Max)
   - GPT Core / Architect: `sub-all/gpt-5.6-luna` (推理深度: Max)
   - Gemini Frontend: `sub-all/gemini-3.7-flash-high`
   - Implementer: `sub-all/gemini-3.7-flash-high`
   - Reviewer: `sub-all/claude-opus-4-6-thinking` (推理深度: xhigh)
3. **权限隔离**：
   - 对 Reviewer 工具层增加 `toolFilter: deny: [write, edit]`，从物理工具层面保证只读审查。
4. **共享记忆机制**：
   - 强制采用 Git 仓库文件与 `.ai/` 状态机作为全局记忆，不依赖临时聊天上下文。

### 结果与收益
- 架构清晰，无第三方依赖包袱。
- 会话具备持续性，通过 `send_message` 支持长期 Agent 状态恢复。
- 完全复用现有 CPA 账号池与 DSH 插件体系。
