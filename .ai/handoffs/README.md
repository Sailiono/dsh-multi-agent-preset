# Agent Handoffs

本目录用于记录不同 Agent 之间的结构化交接单（Handoff Notes）。

## 交接规则
1. 每次一个 Agent 完成某个阶段性交付（如 Architect 完成接口定义、Implementer 完成核心功能等），应在本目录下新建交接文件。
2. 命名规则：`YYYY-MM-DD-<from_agent>-to-<to_agent>-<brief_topic>.md`，例如：
   `2026-08-26-architect-to-implementer-auth-api.md`
3. 下游 Agent 接收任务时，首先阅读该交接单，确认输入产物、接口定义与验收标准。
4. Orchestrator 在合并/验收后，标记交接状态为 `ACCEPTED`。
