# Handoff Note: [任务简述]

- **交接发起人 (From)**: [gpt_core / gemini_frontend / implementer / reviewer]
- **接收方 (To)**: [Orchestrator / implementer / gemini_frontend / reviewer]
- **关联任务 (Task ID)**: TASK-XXX
- **创建时间**: YYYY-MM-DD HH:MM
- **交接状态**: [PENDING / ACCEPTED / REVISED]

---

## 1. 产出概述 (Summary of Deliverables)
- 简述本次完成的主要内容、架构变更或功能实现。

## 2. 关键文件与改动 (Key Files Modified)
- `path/to/file1`: 变更说明 / 接口导出
- `path/to/file2`: 变更说明

## 3. 接口规范与数据契约 (Interface / Contract Specifications)
- 列出下游 Agent 必须遵守的数据结构、API 路由、类型定义或样式类名。

```typescript
// 示例接口
export interface ExampleContract {
  id: string;
  name: string;
}
```

## 4. 依赖与前置条件 (Prerequisites & Constraints)
- 下游开始工作前需要注意的限制、环境变量或已冻结的设计约束。

## 5. 验收标准与测试方法 (Acceptance Criteria & Verification)
- 如何验证下游工作的正确性（命令、测试用例、页面路径等）。
