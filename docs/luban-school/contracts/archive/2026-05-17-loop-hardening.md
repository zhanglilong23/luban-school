# 任务拆解契约 — LOOP-HARDENING-001

**日期**: 2026-05-17
**状态**: 已执行

## WHY

敕令·周天仪和敕令·巡天仪的 SKILL.md 已文档化 `/loop` 指令，但存在三层脱节：命令入口文件未提及 loop 启动步骤、loop prompt 过简（一行字符串，无契约上下文）、无持久化机制。将 loop 从"文档意图"升级为"可执行指令"。

## 任务

| ID | 描述 | 工时 | 风险 | 模式 |
|:---|:---|:---|:---|:---|
| TASK-LOOP-01 | 命令文件补全 loop 启动步骤 | ≤0.5h | 低 | [direct-editing] |
| TASK-LOOP-02 | 周天仪 loop prompt 增强 | ≤0.5h | 低 | [direct-editing] |
| TASK-LOOP-03 | 巡天仪 loop prompt 增强 | ≤0.5h | 低 | [direct-editing] |

## DONE

### TASK-LOOP-01 — 命令文件补全 loop 启动步骤

- [x] `commands/chiling-zhoutian.md` 行为步骤补全第4项：启动定时轮询 `/loop 1m`
- [x] `commands/chiling-xuntian.md` 行为步骤补全第4项：启动定时轮询 `/loop 3m`
- [x] 两项命令文件均注明 loop 为 session-only（会话结束即失效），避免用户误以为 loop 永久运行

### TASK-LOOP-02 — 周天仪 loop prompt 增强

- [x] `skills/chiling-zhoutian/SKILL.md:185` 将一行 prompt 替换为结构化 prompt：
  - 明确定位：`docs/luban-school/shared/status.md`
  - 明确动作链：读取 → 检查 intervention 信号 → 执行干预指令（continue/skip/abort/adjust）
  - 明确频次：每1分钟执行一次
  - 包含心跳写入动作

### TASK-LOOP-03 — 巡天仪 loop prompt 增强

- [x] `skills/chiling-xuntian/SKILL.md:195` 将一行 prompt 替换为结构化 prompt：
  - 明确定位：`docs/luban-school/shared/status.md`
  - 明确动作链：读取 → 检查 deviation/stuck 信号 → 分析 → 写入干预指令
  - 明确频次：每3分钟执行一次
  - 包含 TSK 进度检查（>80% → 写 need_new_tsk 信号）

## DON'T

- 禁止: 改变法器职责边界（不决策、不代劳、不评价、不修改代码）
- 禁止: 修改共享状态文件协议（`docs/luban-school/shared/status.md` 信号定义不变）
- 禁止: 修改五角色 SKILL.md
- 禁止: 修改 CLAUDE.md 触发规则
- 禁止: 引入 durable cron（持久化 loop 不在本次范围，仅增强 prompt + 补齐命令文件）
- 禁止: loop prompt 中写入具体 TASK-ID 或契约内容（loop 是通用机制，契约上下文由 agent 运行时携带）

## 确认记录

- 需求识别: @司南 诊断三层脱节
- 契约拆解: @司南 输出
- 执行编码: @鲁班 交付 [榫卯]
- 质量审查: @狄公 放行
- 终审部署: @墨子 闭环
