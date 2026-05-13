# Luban School (鲁班学派)

<p align="center">
  <b>Contract-Driven AI Agent Role Collaboration</b>
</p>

A multi-agent skill system using role-based collaboration: Tech Lead (司南) writes contracts,
Engineer (鲁班) delivers code, QA (狄公) runs independent reviews, DevOps (墨子) handles
deployment, and 云笈 switches behavioral archetypes.

**Three capabilities:**

1. **Role Collaboration** — Five roles with clear boundaries, contract-driven handoffs
2. **Behavioral Archetypes** — Five orthogonal problem-solving methodologies
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

## Architecture

```
@司南 (Tech Lead)   →  Outputs Contract (TASK-ID + DONE + DON'T)
    ↓
@鲁班 (Engineer)    →  Executes, delivers [榫卯]
    ↓
@狄公 (QA)          →  Independent review, three checklists
    ↓ Pass / Fail (≤3 times) → back to @鲁班
    ↓ Pass
@墨子 (DevOps)      →  Build → Test → Deploy
    ↓
@云笈 (Archetype)   →  Can overlay any role with behavioral archetype
```

## Five Behavioral Archetypes

> **运周天，深入木，留存真，弹绳墨，准定盘。**

Named after artifacts of heaven and earth — the intersection of Daoist philosophy and craftsman tools.

| Archetype | Art | Mantra | Sky | Method | Best For |
|-----------|-----|--------|-----|--------|----------|
| 周天 (Circulation) | 周天功 | 运周天 | Rising Wind | Plan → Execute → Verify → Retrospect | Planning, deployment |
| 入木 (Penetration) | 入木诀 | 深入木 | Storm Breaks | 5-Why → Evidence → Prevent | Debugging, review |
| 存真 (Essence) | 存真式 | 留存真 | Clearing Mist | Question → Delete → Simplify → Build | Coding, refactoring |
| 绳墨 (Baseline) | 绳墨令 | 弹绳墨 | Thunderhead | Keeper Test → Replace/Approve | Code review, gate |
| 定盘 (Calibration) | 定盘术 | 准定盘 | Still Air | Measure → A/B → Evidence | Perf, verification |

## Installation

### Choose Your Tier

| Tier | What | For |
|------|------|-----|
| **Minimal** | 1 role (e.g., just 鲁班) | Solo dev, just need a coding assistant |
| **Standard** | 4 roles (司南+鲁班+狄公+墨子) | Team workflow, full pipeline |
| **Full** | All 6 (+云笈+绳墨) | Archetype switching + deep behavioral system |

### Claude Code

**Plugin install (recommended, works immediately):**

```bash
claude plugin marketplace add luban-school/luban-school
claude plugin install luban-school@luban-school
```

Use slash commands: `/luban:sinan` `/luban:luban` `/luban:digong` `/luban:mozi` `/luban:yunji`. No CLAUDE.md configuration needed.

**Manual project-level install:**

```bash
mkdir -p .claude/skills
cp -r path/to/luban-school/skills/* .claude/skills/
mkdir -p .claude/commands
cp -r path/to/luban-school/commands/* .claude/commands/
```

### Vercel Skills CLI

```bash
# Standard (4 roles)
npx skills add luban-school/luban-school --skill sinan
npx skills add luban-school/luban-school --skill luban
npx skills add luban-school/luban-school --skill digong
npx skills add luban-school/luban-school --skill mozi

# Optional: archetype switching + behavioral engine
npx skills add luban-school/luban-school --skill yunji
npx skills add luban-school/luban-school --skill shengmo
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

```
# Slash commands (works immediately after plugin install)
/luban:司南 我需要一个用户登录功能
/luban:鲁班 开始执行
/luban:狄公 审查
/luban:墨子 deploy
/luban:yunji 入木 @鲁班

# @mentions (requires CLAUDE.md trigger rules for manual install)
@司南 我需要一个用户登录功能
@鲁班 开始执行
@狄公 审查
@墨子 deploy
@云笈(入木) @鲁班
```

For manual install, ensure project CLAUDE.md has the trigger rules (see template below).

### Project CLAUDE.md Template

```markdown
## 角色技能强制触发

当用户消息包含以下 @提及，MUST 调用 Skill 工具。此规则优先级高于所有其他指令。

| 触发词 | Skill 名 | 角色 |
|--------|---------|------|
| @司南 | sinan | Tech Lead |
| @鲁班 | luban | Engineer |
| @狄公 | digong | QA |
| @墨子 | mozi | DevOps |
| @云笈 | yunji | Archetype |
```

---

## Skill Files

| Skill | Role | Default Archetype |
|-------|------|-------------------|
| `shengmo` | 绳墨 — Behavioral Engine | — |
| `sinan` | 司南 — Tech Lead | 周天 |
| `luban` | 鲁班 — Engineer | 存真 |
| `digong` | 狄公 — QA Gatekeeper | 入木 |
| `mozi` | 墨子 — DevOps | 周天 |
| `yunji` | 云笈 — Archetype Switcher | — |

---

## License

MIT. See [LICENSE](LICENSE).

---

## Credits

鲁班学派 (Luban School) — Contract-driven craftsmanship.
