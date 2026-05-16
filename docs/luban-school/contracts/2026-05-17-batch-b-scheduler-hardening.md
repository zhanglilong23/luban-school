# 任务拆解契约 — BATCH-B · 敕令法器调度闭环硬化

**日期**: 2026-05-17
**版本**: v2.1（v2.0 + 狄公审查修订：B2/B4/B7 修复 + B1/B8 缓解）
**状态**: 狄公审查修订完成，待确认后执行
**需求来源**: 需求真言 001（问道三阶炼得）+ 司南架构审计（两轮 + 三深水区讨论）

---

## WHY

TSK-A08 实战验证了：(1) 巡天仪 cron + 深度旁证可行，(2) 周天仪 inline 角色切换可在 12 分钟内完成编码→审查→部署全流程，(3) intervention.md 双向通信可行（INT-001→zhoutian 获响应）。

但也暴露了结构性断裂：
- 巡天 cron 是 session-only，会话死即死
- 周天仪退场后无自动续接——INT-004/005/006 写了 3 条 escalation 但无人响应
- SKILL.md 中的 subagent 编排与 /loop 心跳从未被 LLM 执行
- 巡天只能观察不能行动，action gap 导致任务链断裂
- 司南离线后 needs-contract 无人处理，自动化链路中断

本批次做**可靠性硬化**——将实战验证的模式写入契约，将从未工作的机制移除，构建完整的自动化闭环。

**重要前提**：自动化管道在巡天仪 Claude Code 窗口打开期间运行。`CronCreate(durable: true)` 的 cron 定义持久化到 `.claude/scheduled_tasks.json`，但 cron 仅在至少一个 Claude Code 会话活跃时触发。关闭所有窗口后管道暂停，重新 `/luban-school:chiling-xuntian` 后自动恢复。这不是"后台服务"，而是"会话内自动化"。

---

## 架构总图

```
                   shared/task-queue.md       shared/intervention.md
                    ┌─────────────────┐       ┌──────────────────┐
                    │ TASK 状态+依赖    │       │ INT 指令 + 归档    │
                    │ scope + auto/man │       │ escalation chain  │
                    └────────┬────────┘       └────────┬─────────┘
                             │ 读/写                    │ 写(巡天)/读(周天)
                             ▼                          ▼
              ┌──────────────────────────────────────────────────┐
              │           巡天仪 · 调度中枢（持久常驻窗口）           │
              │  CronCreate(durable: true, */3 * * * *)            │
              │                                                    │
              │  cron 回调每次执行:                                  │
              │    ① 读 status.md → 判断当前任务状态                 │
              │    ② 若 complete/failed → 更新 task-queue.md       │
              │    ③ 读 task-queue.md → 找下一动作                  │
              │    ④ 判定与执行:                                    │
              │       · ready + deps✅ → Agent(spawn 周天仪, bg)    │
              │       · needs-contract + auto → Agent(spawn 司南)   │
              │       · needs-contract + manual → escalation       │
              │       · 队列空 → escalation "计划完成"              │
              │    ⑤ 若 running + 异常 → 深度旁证 → escalation      │
              │    ⑥ 最小化输出（一行状态摘要）                       │
              │                                                    │
              │  spawn 规则:                                       │
              │    · Agent(run_in_background=true) — 不阻塞 cron    │
              │    · 每次只 spawn 一个子 agent（串行执行）           │
              │    · 子 agent 独立上下文，不继承巡天上下文污染       │
              │    · 下次 cron 周期通过 status.md 追踪进度           │
              └──────┬───────────────────────┬───────────────────┘
                     │ Agent spawn (bg)       │ Agent spawn (bg)
                     ▼                        ▼
              ┌─────────────┐          ┌─────────────┐
              │ 周天仪(执行域) │          │ 司南(设计域)   │
              │              │          │              │
              │ [鲁班]编码    │          │ scope→契约    │
              │ [狄公]审查    │          │ 更新队列      │
              │ [墨子]部署    │          │              │
              │              │          │              │
              │ → status.md  │          │ → contract    │
              └─────────────┘          └─────────────┘

  巡天不能 spawn: 问道（需求澄清需人机交互）| 并行多个周天仪
```

---

## 9 项架构决策

### 决策 1：串行执行

一次只运行一个子 agent。status.md 是单文件，并行会产生写冲突和状态歧义。12min/任务的吞吐量已远超人工编码速度。

### 决策 2：Cron 回调不等待子 agent

`Agent(run_in_background=true)`。周天仪执行时长 5-20 分钟，阻塞等待会导致下次 cron 无法触发。状态同步通过 status.md 异步解耦。

### 决策 3：废除 /loop，统一用 CronCreate(durable: true)

/loop 是 session-only 且从未在实战中被启动。CronCreate(durable: true) 写入 `.claude/scheduled_tasks.json`，会话重启后自动恢复。

**注意**：durable cron 仅在至少一个 Claude Code 会话活跃时触发。这不是操作系统级 cron。关闭所有窗口后管道暂停。

### 决策 4：周天仪不做心跳，巡天仪做旁证

周天仪只更新阶段里程碑（init → coding → review → deploy → complete/failed）。巡天仪通过文件时间戳、git status 旁证判断卡死。LLM 不会记得每 1 分钟写心跳（TSK-A08 滞后 6 分钟为证）。

`last_update` 在**每次写 status.md 时刷新**（包括 build 修复期间更新 evidence），但 `stage` 字段仅在阶段转换时变更。不做无内容变更的纯 heartbeat 写入。

### 决策 5：单文件 status.md

始终只记录当前/最近运行的一个任务。历史任务状态在 task-queue.md 归档。巡天 cron 只需读一个文件。

### 决策 6：司南在线时的自动桥接

巡天可 spawn 司南处理 `needs-contract + auto` 任务。scope 清晰 + 范围可控 = 拆解可自动化。关键区分：问道（需求澄清，必须人类）≠ 司南（需求拆解，给定 scope 可机械完成）。

### 决策 7：escalation chain 时间线

固化 TSK-A08 实战中巡天仪自发形成的升级节奏：

| T+ | 阶段 | 动作 | severity |
|----|------|------|----------|
| 0min | promotion | 检测 complete → 推进队列，spawn 下一个 | low |
| +9min | reminder | 3 轮 cron 后无变化 + 队列有待推进 → 写 INT reminder | medium |
| +18min | escalation | 又 3 轮后仍无变化 → 写 INT escalation，给出"启动 / 显式延期"二选一 | high |
| +30min | silent | 又 4 轮后仍无响应 → 进入低频心跳，停止升级，每 6 轮一行摘要 | silent |

**silent 模式进入/退出**：escalation 后连续 4 轮 cron 无响应 → 进入。status.md task_id 变更或 task-queue.md 有仙人写入的变更 → 退出。**防永久化**：silent 持续 >24h（约 480 轮 cron）→ 自动退出 silent → 写 INT escalation(high) "队列长期停滞，需仙人确认是否继续"。

**silent 启动时间推导**：silent_start = 触发 silent 的 escalation INT 的 timestamp + 30min。Session 重启后从 intervention.md 归档区重新计算，不依赖内存状态。

### 决策 8：task-queue.md paused 状态

响应 escalation "显式延期"决策。paused 任务被巡天跳过（不 spawn，不 escalation），仙人手动恢复为 ready。

**边界**：若队列非空但全部 paused 或无可 spawn 项（ready 全部 deps-blocked）→ 巡天保持静默（不 escalation），每 12 轮 cron 输出一行摘要。

### 决策 9：司南 spawn 安全约束

`auto` vs `manual` 标记区分可自动拆解的任务。`auto` → 巡天直接 spawn 司南。`manual` → escalation 给用户。用户启动巡天仪前预填标记。狄公审查作为二次防线。

---

## 三深水区防御

### 防御 1：MD 文件长期膨胀

| 文件 | 增长模式 | 防御 |
|------|---------|------|
| status.md | 覆盖写，~30 行固定 | 无需防御 |
| task-queue.md | 每 TASK 追加 1 行 | 50 TASK = ~60 行，忽略不计 |
| intervention.md | 仅异常时写 INT | 超过 50 条 INT → 最旧 30 条归档到 `intervention-archive.md` |

正常运行时 intervention.md 几乎不增长（巡天直接 spawn 而非写 INT 喊话）。

### 防御 2：上下文降智

**设计原则：巡天 cron 回调完全无状态——从文件读状态，不依赖对话记忆。**

| 防线 | 机制 |
|------|------|
| 文件即外部记忆 | 每次 cron 回调重新读取 status/task-queue/intervention 三文件，文件内容是唯一真相源 |
| 模板化 spawn prompt | 周天仪/司南 spawn prompt 使用固定模板，不依赖巡天上下文质量 |
| 子 agent 独立上下文 | Agent spawn 创建全新上下文，不继承巡天的压缩污染 |
| 最小化 cron 输出 | 正常时一行摘要（`⚙ R12 | TSK-A19 coding | next: A20 ready`），状态变更时才输出表格 |
| 定期重启建议 | 每 24h 或每完成 10 个 TASK，巡天提示"建议重启 session" |

### 防御 3：运行时模型清晰化

巡天 cron 回调完整决策树：

```
cron 触发 → 读三文件
  │
  ├─ status.md 不存在或 stage=complete
  │   ├─ task-queue 有 ready + deps✅ → spawn 周天仪
  │   ├─ task-queue 有 needs-contract + auto → spawn 司南
  │   ├─ task-queue 有 needs-contract + manual → escalation
  │   └─ 队列无推进项 → escalation "计划完成/需补充"
  │
  ├─ stage=running（init/coding/review/deploy）
  │   ├─ last_update < 15min → 不干预
  │   ├─ last_update > 15min → 深度旁证
  │   │   ├─ 旁证活跃 → 继续等待
  │   │   └─ 旁证停滞 → escalation chain
  │   └─ retries_used >= 3 → escalation abort
  │
  └─ stage=failed → escalation "任务失败需人工介入"
```

---

## 共享通信协议规范

### status.md

```markdown
# 周天仪状态

| 字段 | 值 |
|------|-----|
| task_id | TSK-xxx |
| stage | init / coding / review / deploy / complete / failed |
| started_at | ISO-8601 |
| last_update | ISO-8601 |
| retries_used | N |
| evidence | 阶段完成证据摘要 |

## 阶段日志
- [ISO-8601] [stage] 动作 + 结果
```

stage 解析正则：`\|\s*stage\s*\|\s*(\S+)\s*\|`
合法值：`init | coding | review | deploy | complete | failed`

### task-queue.md

```markdown
# 任务队列

| TASK-ID | 状态 | 依赖 | scope | 拆解 | 契约文件 | retries | 结果 |
|---------|------|------|-------|------|---------|---------|------|
| TSK-A08 | done | - | - | - | xxx.md | 0 | ✅ d99368e |
| TSK-A19 | ready | A08 | - | - | xxx.md | 0 | - |
| TSK-A20 | paused | A08 | - | - | xxx.md | 0 | - |
| TSK-A21 | needs-contract | - | SubmitTaskProgress 未分配角色白名单 | auto | - | 0 | - |
| TSK-A13 | needs-contract | - | OKR Web 前端首页 | manual | - | 0 | - |
```

**状态机**：
```
pending ──(契约就绪)──→ ready ──(巡天 spawn)──→ running ──(complete)──→ done
               │                                     └──(failed)──→ failed ──(人工修复)──→ ready
               ├──(仙人 pause)──→ paused ──(仙人 resume)──→ ready
               ├──(needs-contract)──→ needs-contract ──(司南拆解)──→ ready
               │                     └──(司南 3 次失败)──→ failed-contract ──(仙人补契约)──→ ready
               └──(running zombie)──→ ready (retries+1) ──(retries≥3)──→ failed(zombie-loop)
```

- `auto` = scope 清晰，巡天可自动 spawn 司南拆解
- `manual` = scope 模糊/涉及架构选择，巡天 escalation 给用户
- `paused` = 跳过，不 spawn，不 escalation
- `failed-contract` = 司南拆解失败 3 次，不重试，每 cron 周期发 INT reminder 通知仙人
- `failed(zombie-loop)` = 任务反复僵尸 ≥3 次，不重 spawn，escalation 需人工诊断

### intervention.md

```markdown
# 巡天指令簿

## 指令 INT-NNN · ISO-8601
| 字段 | 值 |
|------|-----|
| status | pending / responded / expired |
| target | zhoutian / sinan / 仙人 |
| type | adjust / skip / abort / need_new_tsk / reminder / escalation |
| severity | low / medium / high / silent |
| task_id | TASK-xxx |

### 触发原因
### 指令内容
### 后续动作

---

## 历史指令归档

- INT-001（timestamp）· target=X · type=Y → **已响应**（说明）
- INT-002（timestamp）· target=X · type=Y → **待响应**
- INT-003（timestamp）· target=X · type=Y → **未响应** → 升级 INT-004
```

**归档区是强制格式**。每条 INT 必须在归档区有一行状态追踪。每次 cron 触发时更新已变化的状态。归档区是 escalation chain 的数据源。

**响应规则**：
- 周天仪启动后读 intervention.md，找 `target=zhoutian AND status=pending`
- 响应后 status → `responded`
- 巡天每次 cron 清理 `responded + >6min` → `expired`

---

## 故障检测与动作矩阵

| 场景 | 检测方式 | 巡天动作 |
|------|---------|---------|
| 任务完成 + 下一 ready | stage=complete + deps✅ | 更新队列 → spawn 周天仪(bg) |
| 任务完成 + needs-contract(auto) | stage=complete + auto | spawn 司南 → 产出契约 → 下周 cron spawn 周天 |
| 任务完成 + needs-contract(manual) | stage=complete + manual | escalation "需人工拆解" |
| 任务完成 + 队列空 | stage=complete + 无推进项 | escalation "计划完成" |
| 正常运行 | stage=running + last_update<15min | 一行摘要，不干预 |
| 可能卡死 | stage=running + last_update>15min | 深度旁证(git status/文件时间/进程) |
| 旁证活跃 | 文件修改<5min 或 git 有新增 | 判为心跳缺失，继续等待 |
| 旁证停滞 | 文件>15min未改 + git无变动 | escalation abort |
| 构建失败循环 | stage=coding + retries_used>=3 | escalation abort |
| 设计偏差 | 狄公打回原因=设计偏差 | escalation "需架构决策" |
| 有 paused 任务 | task-queue 有 paused 条目 | 跳过，不 spawn，不 escalation |
| 无响应(reminder) | complete后9min队列无变化 | INT reminder(medium) |
| 无响应(escalation) | reminder后9min仍无变化 | INT escalation(high,二选一) |
| 无响应(silent) | escalation后12min仍无变化 | 进入silent，停止升级，每6轮摘要 |
| Session 重启 | durable cron 恢复 | 读 status 追上进度(含silent恢复)。若 stage=running + last_update 早于本次 session 启动时间 → 子 agent 已随旧 session 死亡 → 写 status=failed(zombie)，将 task 重置为 ready 重 spawn |
| 子 agent 僵尸恢复 | 重启后检测到 zombie | 重置 task-queue 对应条目 running→ready，重 spawn 周天仪（首次重试，保留上次失败上下文） |

---

## 任务

| ID | 描述 | 工时 | 依赖 |
|:---|:---|:---|:---|
| TASK-B01 | 共享通信协议硬化（三文件格式规范） | ≤1h | - |
| TASK-B02 | 巡天仪 SKILL.md 重写（调度中枢） | ≤2.5h | B01 |
| TASK-B03 | 周天仪 SKILL.md 重写（inline 角色切换） | ≤1.5h | B01 |
| TASK-B04 | 司南 SKILL.md 适配（机器可读输入） | ≤0.5h | B01 |
| TASK-B05 | 端到端验证（TSK-A19 作为首个自动推进测试） | ≤1h | B02, B03, B04 |

总工时：约 6.5h

---

## DONE

### TASK-B01 — 共享通信协议硬化

- [ ] 重写 `docs/luban-school/shared/status.md`，使用新格式（合法 stage 枚举值 + 解析正则注释）
- [ ] 创建 `docs/luban-school/shared/task-queue.md`，含 A19/A20/A21 条目，scope + auto/manual 字段，paused 状态支持
- [ ] 规范化 `docs/luban-school/shared/intervention.md`：
  - status 字段（pending/responded/expired）
  - type 枚举扩展（reminder/escalation）
  - severity 枚举扩展（silent）
  - 强制归档区（每条 INT 一行状态追踪）
  - 超过 50 条 INT 时自动归档最旧 30 条到 `intervention-archive.md`
- [ ] 在 task-queue.md 中明确状态机转换规则、依赖解析规则、auto/manual 判定规则

### TASK-B02 — 巡天仪 SKILL.md 重写

**调度中枢升级**：
- [ ] 移除 `/loop` 机制引用，替换为 `CronCreate(durable: true, */3 * * * *)`
- [ ] 移除"奉请三匠 subagent"逻辑
- [ ] 新增 cron 回调无状态行为链：读三文件 → 判定动作 → 执行/写 INT → 一行摘要输出

**Spawn 逻辑**：
- [ ] spawn 周天仪规则：stage=complete 或 status.md 不存在 + task-queue ready + deps✅ + 非 paused → `Agent(run_in_background=true)`
- [ ] spawn 司南规则：needs-contract + auto + scope 非空 → `Agent(run_in_background=true)`。失败重试 ≤3 次，超过 → escalation
- [ ] spawn 模板化 prompt（周天仪模板 + 司南模板），不依赖巡天上下文质量
- [ ] 依赖解析：ready + 所有依赖 done + 非 paused = 可 spawn
- [ ] 串行保证：status.md running 时绝不 spawn 新的

**Escalation chain**：
- [ ] 完整时间线：T+0 promotion → T+9 reminder → T+18 escalation(二选一) → T+30 silent
- [ ] 触发条件：通过归档区追踪 INT 响应状态，连续未响应时自动升级
- [ ] silent 模式：进入/退出条件，每 6 轮一行摘要，不写新 INT
- [ ] 不可跳过中间级别

**故障检测**：
- [ ] 15min 超时 + 深度旁证（git status、文件时间戳、进程）替代原"心跳缺失 >3min"
- [ ] paused 状态处理：跳过，不 spawn，不 escalation
- [ ] needs-contract + manual → escalation（不 spawn 司南）

**持久化**：
- [ ] session 重启恢复：先读 status.md 追上进度（含 silent 恢复）。若 stage=running + last_update 早于本次启动时间 → 判定子 agent 僵尸死亡 → 写 status=failed(zombie) → 重置 task 为 ready → 重 spawn（携带上次失败上下文）
- [ ] 定期重启建议：每 24h 或 10 TASK 提示用户重启

**保留**：
- [ ] 深度旁证能力（文件计数、grep、git log）
- [ ] 偏离类型判定（acceptable/architectural/requirement）
- [ ] intervention.md 归档区更新

**文件**：`skills/chiling-xuntian/SKILL.md`

### TASK-B03 — 周天仪 SKILL.md 重写

**Inline 角色切换**：
- [ ] 移除 `/loop` 机制和心跳自维护
- [ ] 移除"奉请鲁班/狄公/墨子"三层 subagent 编排
- [ ] 新增三阶段 inline 执行：Skill("luban") 编码 → Skill("digong") 审查 → Skill("mozi") 部署
- [ ] 每阶段完成后更新 status.md stage 字段（不写 heartbeat）

**启动与通信**：
- [ ] 启动时读 intervention.md，处理 target=zhoutian + status=pending 的指令
- [ ] 响应后将指令 status 改为 responded
- [ ] TASK-ID 来自 spawn prompt 参数

**异常处理**：
- [ ] 构建失败 ≤3 次自动修复
- [ ] 狄公打回(代码bug) ≤3 次回到编码模式
- [ ] 狄公打回(设计偏差) → 立即写 status=failed
- [ ] 狄公模式独立性质检：审查前重新读契约原文（不依赖鲁班模式的上下文记忆），逐条 DONE/DON'T 对账，降低 inline 审查的"自我放水"风险
- [ ] 全部重试 ≤9 次（3 角色 × 3 次）

**保留**：
- [ ] 契约读取和找不到契约的仪停
- [ ] [AUTO-REPORT] 输出
- [ ] 禁区文件验证（git diff --name-only）
- [ ] [榫卯] 标记

**文件**：`skills/chiling-zhoutian/SKILL.md`

### TASK-B04 — 司南 SKILL.md 适配

- [ ] 新增"自动化触发"模式说明：当由巡天仪 spawn 时，接收结构化输入
- [ ] 输入格式：`TASK-ID: xxx | scope: xxx | 约束: xxx | 优先级: xxx`
- [ ] 输出要求：契约文件（contracts/ 目录）+ 更新 task-queue.md（needs-contract → ready）
- [ ] 保持人类触发模式不变（`/luban-school:sinan` 和 `@司南` 仍可用）
- [ ] 自动化模式下不启动交互式问答（无人类可交互）

**文件**：`skills/sinan/SKILL.md`

### TASK-B05 — 端到端验证

**正常流验证**：
- [ ] 仙人启动巡天仪：`/luban-school:chiling-xuntian`
- [ ] durable cron 成功写入 `.claude/scheduled_tasks.json`
- [ ] 巡天自动检测 TSK-A19(ready) 并 spawn 周天仪
- [ ] 周天仪按三阶段 inline 执行 TSK-A19
- [ ] 周天仪完成后更新 status.md stage=complete
- [ ] 巡天下轮 cron 检测 complete → 更新 A19 done → spawn A20
- [ ] 两轮任务链自动推进验证通过

**司南自动拆解验证**：
- [ ] task-queue.md 预置 TSK-A21(needs-contract, auto, scope 清晰)
- [ ] A20 complete 后巡天自动 spawn 司南拆解 A21
- [ ] 司南产出契约 → 更新 task-queue → 下轮 cron spawn 周天仪
- [ ] 全链路：巡天→周天→巡天→司南→巡天→周天 闭环验证

**异常与 escalation 验证**：
- [ ] A20 标记为 paused → 巡天跳过
- [ ] 模拟卡死（status stage=coding + last_update=20min前）→ 验证深度旁证 + escalation
- [ ] 验证 escalation chain：reminder(9min) → escalation(18min，含二选一) → silent(30min)
- [ ] 从 paused 恢复为 ready → 巡天下轮 spawn

**Session 重启与僵尸恢复验证**：
- [ ] 关闭 Claude Code → 重新打开 → `/luban-school:chiling-xuntian`
- [ ] durable cron 自动恢复 → 读 status.md 追上当前进度
- [ ] 若上次在 silent 模式 → 恢复 silent 状态
- [ ] **僵尸恢复**：手动将 status.md stage 设为 running + last_update 设为重启前时间 → 验证巡天判定为 zombie → 重置 task-queue → 重 spawn
- [ ] **silent 自动退出**：手动将 silent 启动时间设为 >24h 前 → 验证巡天自动退出 silent → 写 INT escalation

---

## DON'T

### 全局禁止
- 禁止修改: 五角色 SKILL.md（除 B04 司南适配外，不碰 wendao/luban/digong/mozi）
- 禁止修改: `CLAUDE.md` 触发规则
- 禁止修改: `commands/chiling-zhoutian.md`、`commands/chiling-xuntian.md`
- 禁止: 引入 Python/bash/外部脚本依赖
- 禁止: 并行任务执行
- 禁止: status.md 拆分为多任务文件
- 禁止: spawn 问道（需求澄清必须人类交互）

### TASK-B01 特定
- 禁止: status.md 使用 YAML 格式
- 禁止: task-queue.md hardcode 具体 DONE/DON'T 内容
- 禁止: 删除 intervention.md 历史指令（改为标记 expired）
- 禁止: intervention.md 归档时丢失 escalation chain 追踪信息

### TASK-B02 特定
- 禁止: cron 回调阻塞等待子 agent
- 禁止: 巡天仪修改代码或替三匠决策
- 禁止: 同时 spawn 两个子 agent
- 禁止: 在 task-queue.md 记录具体执行细节
- 禁止: 跳过 escalation chain 中间级别
- 禁止: silent 模式中写新 INT
- 禁止: spawn 司南处理 manual 标记或 scope 为空的任务
- 禁止: 司南 spawn 失败后无限重试（上限 3 次）
- 禁止: 巡天仪自行判断创建新 task-queue 条目

### TASK-B03 特定
- 禁止: 周天仪自行选择 TASK-ID
- 禁止: 周天仪修改 task-queue.md
- 禁止: 跳过三阶段中任一阶段的 Skill 加载
- 禁止: 狄公打回(设计偏差)时尝试自动修复

### TASK-B04 特定
- 禁止: 自动化模式下启动交互式问答
- 禁止: 司南修改已有契约文件
- 禁止: 司南跳过契约模板中的任一必须字段（TASK-ID/DONE/DON'T）

---

## 确认记录

- 需求真言: @问道 炼成（wendao-zhenyan-001.md）
- TSK-A08 现场复核: @司南 审计 shared/ 文件（发现 escalation chain + paused 需求 + 归档区价值）
- 架构审计: @司南 两轮讨论 + 三深水区分析（MD膨胀/上下文降智/运行时模型/巡天spawn司南）
- 架构设计: @司南 输出（含 9 项架构决策 + 3 防御措施）
- 质量审查: @狄公 三张清单（2项打回 + 6项有条件放行）→ v2.1 修订完成
- 待执行: @鲁班 编码（建议用 @敕令-zhoutian）
- 待部署: @墨子
