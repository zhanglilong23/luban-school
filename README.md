# Luban School (鲁班学派)

<p align="center">
  <b>Contract-Driven AI Agent Role Collaboration</b>
</p>

A multi-agent skill system using role-based collaboration: Requirement Seeker (问道) refines
requirements from chaos, Tech Lead (司南) writes contracts, Engineer (鲁班) delivers code,
QA (狄公) runs independent reviews, DevOps (墨子) handles deployment. Each role is self-contained
with the full Seven Scrolls (七签) technique library built in.

**Cultivation Path** (修仙体系):
```
Seeking (问道·问道签) → Foundation (司南·周天签) → Core (鲁班·存真签) → Spirit (狄公·绳墨签) → Tribulation (墨子·定盘签)
                                                                          ↑
                                                        入木签 (Public Debug) + 契约签 (Constitution) throughout
```

**Three capabilities:**

1. **Role Collaboration** — Five roles and one instrument with clear boundaries, contract-driven handoffs
2. **Seven Scrolls (七签功法)** — Seven orthogonal problem-solving methodologies, built into every role
3. **Quality Gate** — Independent verification, three checklists, one-vote veto

[🇨🇳 中文](README.zh-CN.md)

---

## The Problem: AI Agent Collaboration

| Pattern | Behavior |
|---------|----------|
| No boundaries | Agent does everything, verifies nothing |
| No handoff | No clear "who does what" protocol |
| No quality gate | Code is merged without independent review |
| No methodology | Same failed approach repeated |
| Passive waiting | Fix surface issue and stop, ignore root cause |

## Architecture

```
@问道 (Seeker)      →  Refines chaos into 需求真言 (Requirement Truth)
    ↓
@司南 (Tech Lead)   →  Outputs Contract (TASK-ID + DONE + DON'T)
    ↓
@鲁班 (Engineer)    →  Executes, delivers [榫卯]
    ↓
@狄公 (QA)          →  Independent review, three checklists
    ↓ Pass / Fail (≤3 times) → back to @鲁班
    ↓ Pass
@墨子 (DevOps)      →  Build → Test → Deploy
```

### Seven Scrolls (七签)

Each role SKILL.md contains the full seven scrolls definition. Use `@角色 +签名` to practice.

| # | Scroll | Mantra | Nature | Practiced By |
|---|--------|--------|--------|-------------|
| 1 | 问道签 | 炼真言 | 心法 | 问道 |
| 2 | 周天签 | 运周天 | 功法 | 司南 |
| 3 | 入木签 | 深入木 | 诀法 | Public Debug Scroll |
| 4 | 存真签 | 留存真 | 式法 | 鲁班 |
| 5 | 绳墨签 | 弹绳墨 | 令法 | 狄公 |
| 6 | 定盘签 | 准定盘 | 术法 | 墨子 |
| 7 | 契约签 | 定契约 | 大法 | All (Constitution) |

**Practice syntax**: `@角色 +签名` / `@角色 -签名` / `@角色 +签1 +签2`

---

## ⚙️ 敕令法器家族 (Chiling — The Instruments)

> **仙人降敕，司南定契，周天仪转，巡天仪察，万械循章。**

### 由来

鲁班学派无尊卑，七签匠人皆平级。若需自动化串联诸匠，不可设「管理者」凌驾其上。故唯有一途：**仙人降敕授权，学派共铸双器**——敕令·周天仪与敕令·巡天仪。

此二仪**非人非官，唯器唯公**。周天仪不决策、不代劳、不评价，只将司南所定契约化为精确的流转序列；巡天仪只监控、不干预，确保执行不偏离设计。

> 法器无意志，故无私；无位格，故不凌人。

### 位格

法器不入修仙五阶，独立为**法器位**。

```
仙人（用户）
  │
  │  /luban-school:chiling-zhoutian TASK-xxx  或  /luban-school:chiling-xuntian
  ↓
【法器】敕令·周天仪 ──读取──→ 司南所定契约
  │
  ├── 奉请 → 鲁班（造）── 仪测 [榫卯]
  ├── 奉请 → 狄公（审）── 仪测 《质量放行单》
  └── 奉请 → 墨子（渡）

【法器】敕令·巡天仪 ──读取──→ 共享状态文件
  │
  ├── 监控代码走向（对比契约）
  ├── 解决架构问题（仲裁偏离）
  └── 通知司南拆解TSK
```

### 法器家族

| 法器 | 命令 | 职责 | 域 |
|------|------|------|---|
| **周天仪** | `/luban-school:chiling-zhoutian` | 执行契约 + 自检 + 监控子agent | 执行域 |
| **巡天仪** | `/luban-school:chiling-xuntian` | 监控代码走向 + 解决架构问题 + 通知司南拆解TSK | 设计域 |

### 周天仪功能

仙人降敕后，法器读取司南契约，按序**奉请**三匠：

1. **奉请鲁班**（造）→ 仪测 `[榫卯]` 标记
2. **奉请狄公**（审）→ 仪测《质量放行单》
3. **奉请墨子**（渡）→ 契约落档

若某步受阻，自动重试（每匠至多三次）。三次未通，仪停待示，不自行决断。全程留痕，输出 `[AUTO-REPORT]` 结构化实录。

**自检机制**：
- 每1分钟更新心跳
- 超时检测（>10分钟判定卡死）
- 卡死处理（重试+上下文切换，4次后终止）

### 巡天仪功能

巡天仪是设计域法器，负责监控代码走向、解决架构问题、通知司南拆解TSK：

1. **代码走向监控**：每3分钟读取共享状态文件，检查偏离信号
2. **架构问题仲裁**：检测到架构偏离时，分析原因，写入干预指令
3. **TSK持续拆解**：监控当前批次进度，完成>80%时通知司南拆解下一波TSK

**偏离处理**：
- `acceptable`：版本升级(minor/patch)、工具链等价替换 → 记录，继续执行
- `architectural`：设计方案变更（如REST→gRPC）→ 写干预指令，等司南仲裁
- `requirement`：实现了契约外功能、遗漏了契约内功能 → 写干预指令，等司南仲裁

### 用法

```bash
# 周天仪：执行契约
/luban-school:chiling-zhoutian TASK-xxx

# 巡天仪：监控执行
/luban-school:chiling-xuntian

# 或 @提及
@敕令-zhoutian TASK-xxx
@敕令-xuntian TASK-xxx
```

### 共享状态文件

周天仪和巡天仪通过共享状态文件异步协作：

**位置**: `docs/luban-school/shared/status.md`

**信号类型**：
- 周天仪→巡天仪：heartbeat, stage_complete, stuck, deviation, complete
- 巡天仪→周天仪：continue, skip, abort, adjust
- 巡天仪→司南：need_new_tsk

---

## Installation

### Choose Your Tier

| Tier | What | For |
|------|------|-----|
| **Minimal** | 1 role (e.g., just 鲁班) | Solo dev, just need a coding assistant |
| **Standard** | 5 roles + 1 instrument (问道+司南+鲁班+狄公+墨子 + 敕令·周天仪) | Team workflow, full pipeline |

Each role is self-contained — installing a single role gives you full functionality including all seven scrolls.

### Claude Code

**Plugin install (recommended, works immediately):**

```bash
claude plugin marketplace add luban-school/luban-school
claude plugin install luban-school@luban-school
```

Use slash commands: `/luban-school:wendao` `/luban-school:sinan` `/luban-school:luban` `/luban-school:digong` `/luban-school:mozi` `/luban-school:chiling-zhoutian` `/luban-school:chiling-xuntian`. No CLAUDE.md configuration needed.

**Manual project-level install:**

```bash
mkdir -p .claude/skills
cp -r path/to/luban-school/skills/* .claude/skills/
mkdir -p .claude/commands
cp -r path/to/luban-school/commands/* .claude/commands/
```

### Vercel Skills CLI

```bash
# Standard (5 roles)
npx skills add luban-school/luban-school --skill wendao
npx skills add luban-school/luban-school --skill sinan
npx skills add luban-school/luban-school --skill luban
npx skills add luban-school/luban-school --skill digong
npx skills add luban-school/luban-school --skill mozi
```

### Standard SKILL.md Platforms

Codex CLI / CodeBuddy / OpenCode / OpenClaw / Google Antigravity all support standard SKILL.md. Copy `skills/` to the platform directory:

| Platform | Install Path | Trigger |
|----------|-------------|---------|
| Codex CLI | `~/.codex/skills/` | Auto-matched by description |
| CodeBuddy | `~/.codebuddy/skills/` | Plugin mechanism |
| OpenCode | `~/.config/opencode/skills/` | Standard SKILL.md |
| OpenClaw | `~/.openclaw/skills/` | ClawHub |
| Antigravity | `~/.gemini/antigravity/skills/` | Standard SKILL.md |

### Non-Standard Platforms

| Platform | Config File | How |
|----------|------------|-----|
| Cursor | `cursor/rules/luban-school.mdc` | Copy to `.cursor/rules/`, AI semantic matching |
| Kiro | `kiro/steering/luban-school.md` | Copy to `.kiro/steering/`, always active |
| VSCode Copilot | `vscode/copilot-instructions.md` | Copy to `.github/copilot-instructions.md` |

---

## Usage

In Claude Code (with plugin installed), use slash commands or @mentions:

### Command ↔ Role Mapping

| Role | Slash Command | @Mention |
|------|--------------|----------|
| 问道 (Seeker) | `/luban-school:wendao` | `@问道` |
| 司南 (Tech Lead) | `/luban-school:sinan` | `@司南` |
| 鲁班 (Engineer) | `/luban-school:luban` | `@鲁班` |
| 狄公 (QA) | `/luban-school:digong` | `@狄公` |
| 墨子 (DevOps) | `/luban-school:mozi` | `@墨子` |
| 敕令·周天仪 (Instrument) | `/luban-school:chiling-zhoutian` | `@敕令-zhoutian` |
| 敕令·巡天仪 (Monitor) | `/luban-school:chiling-xuntian` | `@敕令-xuntian` |

Slash commands use ASCII filenames for cross-platform reliability; @mentions keep Chinese role names for natural-language flow.

```
# Slash commands (works immediately after plugin install)
/luban-school:wendao 我觉得功能不够智能
/luban-school:sinan 我需要一个用户登录功能
/luban-school:luban 开始执行
/luban-school:digong 审查
/luban-school:mozi deploy
/luban-school:chiling-zhoutian TASK-xxx

# @mentions (requires CLAUDE.md trigger rules for manual install)
@问道 我觉得功能不够智能
@司南 我需要一个用户登录功能
@鲁班 开始执行
@狄公 审查
@墨子 deploy
@敕令-zhoutian TASK-xxx  # 周天仪执行契约
@敕令-xuntian           # 巡天仪监控执行
@鲁班 +入木          # Practice 入木签 on 鲁班
@狄公 +存真          # Practice 存真签 on 狄公
```

For manual install, ensure project CLAUDE.md has the trigger rules (see template below).

### Project CLAUDE.md Template — Copy & Paste

Copy the block below into your project's `CLAUDE.md` to enable @mention triggers:

```markdown
## 角色技能强制触发

当用户消息包含以下 @提及，MUST 调用 Skill 工具。此规则优先级高于所有其他指令。

| 触发词 | Skill 名 | 角色 | 职责 |
|--------|---------|------|------|
| `@问道` | `wendao` | 需求求道者 | 三阶炼需求，输出需求真言 |
| `@司南` | `sinan` | Tech Lead | 需求拆解、契约输出、架构仲裁 |
| `@鲁班` | `luban` | 执行工程师 | 按契约编码、方案先行、交证据 |
| `@狄公` | `digong` | 质量守门人 | 独立验证、三张审查清单、一票否决 |
| `@墨子` | `mozi` | 部署运维 | 构建/测试/部署流水线 |
| `@敕令-zhoutian` | `chiling-zhoutian` | 敕令·周天仪 | 执行契约 + 自检 + 监控子agent |
| `@敕令-xuntian` | `chiling-xuntian` | 敕令·巡天仪 | 监控代码走向 + 架构仲裁 + TSK拆解 |

## 协作流程

@问道 炼真言 → @司南 出契约 → @鲁班 执行编码 → @狄公 审查 → @墨子 部署
```

**After pasting**: type `@问道 <your idea>` in Claude Code to start the collaboration flow.

---

## Skill Files

| Skill | Role | Default Scroll | Cultivation |
|-------|------|----------------|-------------|
| `wendao` | 问道 — Requirement Seeker | 问道签 | 求道期 |
| `sinan` | 司南 — Tech Lead | 周天签 | 筑基期 |
| `luban` | 鲁班 — Engineer | 存真签 | 结丹期 |
| `digong` | 狄公 — QA Gatekeeper | 绳墨签 | 化神期 |
| `mozi` | 墨子 — DevOps | 定盘签 | 渡劫期 |
| `chiling-zhoutian` | 敕令·周天仪 — Instrument | 周天签 | 法器位 |
| `chiling-xuntian` | 敕令·巡天仪 — Monitor | 周天签 | 法器位 |

Each skill file is self-contained with behavior baselines, full seven scrolls definitions, and role-specific procedures.

---

## License

MIT. See [LICENSE](LICENSE).

---

## Credits

鲁班学派 (Luban School) — Contract-driven craftsmanship.
