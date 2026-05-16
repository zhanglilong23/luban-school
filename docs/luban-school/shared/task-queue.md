# 任务队列

> 巡天仪据此判断下一动作。状态机：pending → ready → running → done/failed。paused 跳过。

| TASK-ID | 状态 | 依赖 | scope | 拆解 | 契约文件 | 结果 |
|---------|------|------|-------|------|---------|------|
| BATCH-B | running | - | - | - | 2026-05-17-batch-b-scheduler-hardening.md | - |
| TSK-A19 | ready | A08 | - | - | 2026-05-16-tsk-a19-approval-gate-state-machine.md | - |
| TSK-A20 | ready | A08 | - | - | 2026-05-17-tsk-a20-permission-gate.md | - |
| TSK-A21 | needs-contract | - | SubmitTaskProgress 未分配到任何角色白名单（A08 证据: RolePermissionMatrix.cs grep 0匹配） | auto | - | - |
| TSK-A13 | needs-contract | A12 | OKR Web 生产前端首页（仙人 D1: 前端建在本地 frontend/xxx/） | manual | - | - |

## 状态机规则

- `pending` → (契约就绪) → `ready` → (巡天 spawn 周天) → `running` → (complete) → `done`
- `running` → (failed) → `failed` → (人工修复) → `ready`
- `ready` → (仙人/巡天 pause) → `paused` → (仙人 resume) → `ready`
- `pending` → (needs-contract) → `needs-contract` → (司南拆解) → `ready`

## 依赖解析

- `ready` + 所有依赖 TASK 状态为 `done` + 非 `paused` → 可 spawn
- `needs-contract` + `auto` → 巡天 spawn 司南（失败重试 ≤3 次）
- `needs-contract` + `manual` → 巡天 escalation 给仙人
- `paused` → 跳过，不 spawn，不 escalation
