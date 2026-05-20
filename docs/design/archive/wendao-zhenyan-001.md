# 需求真言 001 · 敕令法器调度闭环硬化

> 问道三阶炼得，转交 @司南 拆解。

## 真言

**I want** 巡天仪升级为 durable cron 持久调度中枢，周天仪以 inline 角色切换方式执行三阶段交付，两法器通过 shared/ 文件系统异步通信，自动推进任务链直到队列清空。
**So that** 问道→司南→法器执行的自动化闭环不依赖 LLM 会话存续，单任务的交付质量不低于 TSK-A08 已证明的水平（12min / 110 tests pass / 0 禁区触碰）。

---

## 基石证据（来自 TSK-A08 实战）

| 已验证可行的 | 已验证不可行的 |
|-------------|---------------|
| Cron + cron 回调执行巡检 | LLM 每 1 分钟更新心跳（6 分钟滞后） |
| 周天仪 inline 四角色切换完成三阶段 | 周天仪 spawn 三匠 subagent（从未触发） |
| intervention.md 双向通信闭环 | YAML 信号协议追加写入（LLM 用简单表格替代） |
| 巡天仪旁证验证（文件计数、grep、git log） | /loop 自维护心跳（从未启动） |
| Agent spawn 单层子 agent | 两层嵌套 spawn（不可靠） |

---

## 目标架构

```
                 ┌─ 任务队列 ──────────────────────┐
                 │  shared/task-queue.md            │
                 │  TASK-A19 ready (dep: A08 ✅)     │
                 │  TASK-A20 ready (dep: A08 ✅)     │
                 │  TASK-A21 ready (dep: none)       │
                 └──────────────────────────────────┘
                        ↑ 读（判断下一个 ready）
                        │
┌─────────────────────────────────────────────────────┐
│  巡天仪 · 持久调度中枢（一个常驻窗口）                   │
│                                                     │
│  CronCreate(durable: true, 每 3min)                  │
│       │                                             │
│       ├── 读 status.md                              │
│       ├── 读 task-queue.md                          │
│       │                                             │
│       ├── stage=complete? → 标记当前 TASK done       │
│       │   └── 找下一个 ready 任务                    │
│       │       ├── 有 ready → Agent(spawn 周天仪)     │
│       │       └── 无 ready + 需新契约 → 通知仙人      │
│       │                                             │
│       ├── stage=running + 心跳缺失? → 深度旁证        │
│       │   └── 确实卡死 → 写 abort 指令               │
│       │                                             │
│       └── 发现偏离? → 写 intervention.md 指令        │
│                                                     │
│  子 agent（周天仪，单层，执行单个 TASK）：              │
│       │                                             │
│       ├── [鲁班模式] 写代码 + 构建 + 测试              │
│       ├── [狄公模式] 三张清单 + 放行/打回              │
│       ├── [墨子模式] 构建验证 + commit                 │
│       └── 写 status.md stage=complete                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 四个改动维度

### 维度 1：巡天仪 SKILL.md 升级

**改什么**：从"只读监控者"升级为"调度中枢"。

| 当前 | 改为 |
|------|------|
| `/loop 3m`（从未启动） | `CronCreate(durable: true, 每 3min)` |
| 只写 INT 信号等人看 | cron 回调中主动 `Agent(spawn)` 周天仪 |
| 进度 >80% → 通知司南 | 任务 complete → 直接查 task-queue.md → 有 ready 就 spawn |
| 偏离全等司南仲裁 | acceptable → 记录继续 / architectural+requirement → 写 INT + 暂停对该 TASK 的推进 |

### 维度 2：周天仪 SKILL.md 务实化

**改什么**：承认 inline 角色切换模式，移除三层 subagent 编排。

| 当前 | 改为 |
|------|------|
| spawn 鲁班 → spawn 狄公 → spawn 墨子 | Skill 加载切换：先 `Skill("luban")` 编码 → `Skill("digong")` 审查 → `Skill("mozi")` 部署 |
| `/loop 1m` 心跳（从未启动） | 每阶段完成后更新 status.md 即可（巡天仪旁证兜底） |
| YAML 信号协议 | markdown 表格 + intervention.md（已验证可用） |
| 三匠重试各 3 次 | 保留重试逻辑但降低复杂度：build 失败 → 自动修 → 3 次失败才仪停 |

### 维度 3：新建 task-queue.md 协议

新文件 `docs/luban-school/shared/task-queue.md`：

```markdown
# 任务队列

| TASK-ID | 状态 | 依赖 | 契约文件 | 结果 |
|---------|------|------|---------|------|
| TSK-A08 | done | - | 2026-05-16-tsk-a08-tool-wrapping.md | ✅ 110 tests, d99368e |
| TSK-A19 | ready | A08 | 2026-05-16-tsk-a19-approval-gate-state-machine.md | - |
| TSK-A20 | ready | A08 | 2026-05-17-tsk-a20-permission-gate.md | - |
| TSK-A21 | ready | - | (待司南拆解) | - |
```

巡天仪读取规则：
- `ready` + 依赖全部 `done` → 可 spawn
- `ready` + 依赖未完成 → 等待
- 无 `ready` 任务 → 通知仙人

### 维度 4：intervention.md 保留并规范化

已证明有效的通信格式，无需大改。补充：
- 每条指令增加 `status: pending / responded / expired`
- 周天仪启动后必须读取 intervention.md 中 target=zhoutian 的 pending 指令
- 巡天仪每次 cron 触发时清理已响应的指令

---

## 不改的（已验证足够好）

- **status.md** 当前格式：简单 markdown 表格，不强制 YAML
- **周天仪 inline 执行模式**：不恢复 subagent spawn
- **巡天仪深度巡检方式**：旁证（grep/git/filesystem）优于信任单一 status 字段
- **Cron 间隔**：巡天 3 分钟足够，周天仪不需要自己的心跳循环

---

## 验收标准

1. 巡天仪 cron 写入 `.claude/scheduled_tasks.json`（durable 持久化）
2. 周天仪完成 TASK 后，下次 cron 触发时巡天仪能自动检测并 spawn 下一个周天仪
3. task-queue.md 中的依赖解析正确（A20 依赖 A08，A08 done 后才 spawn A20）
4. 任务链异常中断时（如构建失败 3 次），巡天仪输出仪停报告并等待人工介入
5. 巡天仪窗口关闭重启后，durable cron 自动恢复，从 status.md 追上当前进度

---

> 问道出品 · 2026-05-17 · 转交 @司南 拆解
