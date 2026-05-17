# 任务队列

> 巡天仪据此判断下一动作。状态机：pending → ready → running → done/failed。paused 跳过。

| TASK-ID | 状态 | 依赖 | scope | 拆解 | 契约文件 | retries | 结果 |
|---------|------|------|-------|------|---------|---------|------|
| BATCH-B | running | - | - | - | 2026-05-17-batch-b-scheduler-hardening.md | 0 | - |
| TSK-A19 | ready | A08 | - | - | 2026-05-16-tsk-a19-approval-gate-state-machine.md | 0 | - |
| TSK-A20 | ready | A08 | - | - | 2026-05-17-tsk-a20-permission-gate.md | 0 | - |
| TSK-A21 | needs-contract | - | SubmitTaskProgress 未分配到任何角色白名单（A08 证据: RolePermissionMatrix.cs grep 0匹配） | auto | - | 0 | - |
| TSK-A13 | needs-contract | A12 | OKR Web 生产前端首页（仙人 D1: 前端建在本地 frontend/xxx/） | manual | - | 0 | - |

## 状态机规则

- `pending` → (契约就绪) → `ready` → (巡天 spawn 周天) → `running` → (complete) → `done`
- `running` → (failed) → `failed` → (人工修复) → `ready`
- `running` → (zombie) → `ready`（retries+1。若 retries≥3 → `failed(zombie-loop)` → escalation）
- `ready` → (仙人/巡天 pause) → `paused` → (仙人 resume) → `ready`
- `pending` → (needs-contract) → `needs-contract` → (司南拆解) → `ready`
- `needs-contract` → (司南 spawn 失败 3 次) → `failed-contract` → (仙人补契约) → `ready`

## 依赖解析

- `ready` + 所有依赖 TASK 状态为 `done` + 非 `paused` → 可 spawn
- `needs-contract` + `auto` → 巡天 spawn 司南（失败重试 ≤3 次，超过 → `failed-contract`）
- `needs-contract` + `manual` → 巡天 escalation 给仙人
- `failed-contract` → 不 spawn，不重试，每 cron 周期写 INT reminder 通知仙人
- `paused` → 跳过，不 spawn，不 escalation
- `failed(zombie-loop)` → 不 spawn，写 INT escalation "任务反复僵尸需人工诊断"

## retries 列语义

`retries` 仅计数 **task 级别的重 spawn**（zombie 恢复 + 司南 spawn 失败重试），不计数周天仪内部的构建/测试重试（那在 status.md `retries_used` 字段中）。

---

## 巡天独立验证清单

> 每次 task 标 `completed` 前，巡天必须独立验证以下四件（不依赖周天仪自报）：

| # | 验证项 | 命令 | 通过标准 |
|---|--------|------|---------|
| 1 | commit hash 存在 | `git log --oneline -5` | 对应 hash 在最近 commit 中 |
| 2 | 变更范围在白名单 | `git diff <prev>..<curr> --name-only` | 所有文件在契约白名单内 |
| 3 | 红线区零越界 | grep 红线区关键词 | 0 匹配 |
| 4 | commit msg 含验收数据 | `git log -1 --format=%B` | 含测试数 / 偏离清单 / 红线区证据 |

未通过任一条 → 不标 completed，记录偏离，必要时回写 status.md。
