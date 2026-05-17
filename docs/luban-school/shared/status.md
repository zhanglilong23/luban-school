# 周天仪状态 · IDLE

| 字段 | 值 |
|------|-----|
| task_id | IDLE |
| stage | idle |
| started_at | — |
| last_update | 2026-05-17T12:30:00Z |
| retries_used | 0 |
| evidence | — |

## 阶段日志
- [idle] 无进行中批次，等待下一契约

<!-- stage 解析正则: \|\s*stage\s*\|\s*(\S+)\s*\| -->
<!-- 合法值: init | coding | review | deploy | complete | failed | idle -->
