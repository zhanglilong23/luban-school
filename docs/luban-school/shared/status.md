# 共享状态文件 — 敕令法器家族通信协议

> 本文件是周天仪与巡天仪的异步通信桥梁。所有信号以追加方式写入，保留完整历史。

## 文件格式

```yaml
# 状态文件结构
version: "1.0"
last_updated: "ISO-8601 时间戳"

# 当前执行状态
execution:
  task_id: "TASK-xxx"
  status: "running | stuck | complete | aborted"
  current_stage: "coding | review | deploy"
  started_at: "ISO-8601 时间戳"
  heartbeat: "ISO-8601 时间戳"  # 每1分钟更新

# 信号日志（追加写入）
signals:
  - type: "heartbeat | stage_complete | stuck | deviation | complete | continue | skip | abort | adjust | need_new_tsk"
    timestamp: "ISO-8601 时间戳"
    source: "zhoutian | xuntian"
    data: {}
```

## 信号定义

### 周天仪 → 巡天仪

| 信号 | 字段 | 频率 | 说明 |
|------|------|------|------|
| `heartbeat` | task_id, stage, elapsed_min, subagent_status | 每1分钟 | 心跳，证明周天仪还活着 |
| `stage_complete` | task_id, stage, result, evidence, duration_min | 阶段完成时 | 阶段完成通知 |
| `stuck` | task_id, stage, reason, retry_count, elapsed_min | 卡死时 | 卡死求助信号 |
| `deviation` | task_id, type(expected/acceptable/architectural/requirement), expected, actual, reason | 偏离时 | 偏离检测信号 |
| `complete` | task_id, report, total_duration_min | 任务完成时 | 任务完成通知 |

### 巡天仪 → 周天仪

| 信号 | 字段 | 频率 | 说明 |
|------|------|------|------|
| `continue` | task_id, instruction, context | 需要时 | 继续执行指令 |
| `skip` | task_id, stage, reason, next_stage | 需要时 | 跳过阶段指令 |
| `abort` | task_id, reason, final_status | 需要时 | 终止任务指令 |
| `adjust` | task_id, field, old_value, new_value, reason | 需要时 | 调整参数指令 |

### 巡天仪 → 司南

| 信号 | 字段 | 频率 | 说明 |
|------|------|------|------|
| `need_new_tsk` | current_tsk_count, completed_tsk, reason, suggested_scope | 需要时 | 通知司南拆解新TSK |

## 卡死判定标准

| 条件 | 阈值 | 判定者 | 动作 |
|------|------|--------|------|
| 单阶段执行时间 | >10 分钟 | 周天仪自检 | 写 `stuck` 信号 |
| 心跳缺失 | >3 分钟无更新 | 巡天仪 | 写 `abort` 或 `skip` 指令 |
| 同一阶段重试 | >3 次 | 巡天仪 | 写 `abort` 指令 |

## 偏离处理规则

| 偏离类型 | 判定条件 | 处理方式 |
|---------|---------|---------|
| `acceptable` | 版本升级(minor/patch)、工具链等价替换、实现细节调整但满足DONE | 记录，继续执行 |
| `architectural` | 设计方案变更（如REST→gRPC）、核心逻辑变更 | 等巡天仪仲裁 |
| `requirement` | 实现了契约外功能、遗漏了契约内功能 | 等巡天仪仲裁 |

## 卡死处理方案（重试+上下文切换）

| 重试次数 | 上下文切换 | 效果 |
|---------|-----------|------|
| 第1次 | 带上失败上下文（卡在哪里、错误信息） | 鲁班知道上次卡在哪里，避免重复 |
| 第2次 | 换一种实现方式（如不用搜索，直接用已知信息） | 换思路，避免同样的死循环 |
| 第3次 | 简化需求（只实现核心功能，跳过边界情况） | 保证核心功能完成 |
| 第4次 | 终止，报告用户 | 请求人工决策 |

**简化需求的边界**：
1. 核心功能完整
2. 代码可编译
3. 基本流程可运行

## 使用示例

### 心跳更新（周天仪每分钟写入）

```yaml
signals:
  - type: "heartbeat"
    timestamp: "2026-05-14T20:30:00Z"
    source: "zhoutian"
    data:
      task_id: "TASK-CH01"
      stage: "coding"
      elapsed_min: 5
      subagent_status: "running"
```

### 卡死信号（周天仪检测到卡死）

```yaml
signals:
  - type: "stuck"
    timestamp: "2026-05-14T20:35:00Z"
    source: "zhoutian"
    data:
      task_id: "TASK-CH01"
      stage: "coding"
      reason: "鲁班在搜索 ToolApprovalRequestContent 时陷入循环"
      retry_count: 1
      elapsed_min: 10
```

### 干预指令（巡天仪响应卡死）

```yaml
signals:
  - type: "continue"
    timestamp: "2026-05-14T20:35:30Z"
    source: "xuntian"
    data:
      task_id: "TASK-CH01"
      instruction: "跳过搜索，直接用已知信息编码"
      context: "已知 ToolApprovalRequestContent 在 WorkflowSession.cs 第640行"
```

### 偏离信号（周天仪检测到偏离）

```yaml
signals:
  - type: "deviation"
    timestamp: "2026-05-14T20:40:00Z"
    source: "zhoutian"
    data:
      task_id: "TASK-CH01"
      type: "acceptable"
      expected: "nuget 1.0.0"
      actual: "nuget 1.1.0"
      reason: "minor版本升级，无破坏性变更"
```

### TSK拆解通知（巡天仪通知司南）

```yaml
signals:
  - type: "need_new_tsk"
    timestamp: "2026-05-14T21:00:00Z"
    source: "xuntian"
    data:
      current_tsk_count: 5
      completed_tsk: ["TASK-CH01", "TASK-CH02", "TASK-CH03"]
      reason: "当前批次即将完成，需要拆解下一波TSK"
      suggested_scope: "TASK-CH06至TASK-CH10"
```
