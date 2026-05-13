# 鲁班学派

<p align="center">
  <b>契约驱动的 AI Agent 角色分工协作体系</b>
</p>

五角色、五原型、一份契约。司南拆需求定契约、鲁班编码闭环交付、狄公独立质量审查、墨子部署运维、云笈行为原型切换。

**三重能力：**

1. **角色分工协作** — 五角色边界清晰，契约驱动交付
2. **行为原型体系** — 五个正交方法论，按场景切换
3. **质量门禁** — 独立验证、三张清单、一票否决

**[🇺🇸 English](README.md)**

---

## 问题：AI Agent 协作的五大痛点

| 痛点 | 表现 |
|------|------|
| 边界模糊 | Agent 什么都做，什么都不验证 |
| 缺乏交接 | 没有"谁做什么"的明确协议 |
| 缺乏审查 | 代码不经独立 review 直接合入 |
| 方法僵化 | 同一失败方案反复尝试 |
| 被动等待 | 修完表面问题就停下 |

## 架构

```
@司南 (Tech Lead)   →  输出《任务拆解契约》(TASK-ID + DONE + DON'T)
    ↓
@鲁班 (Engineer)    →  按契约编码，交付 [榫卯]
    ↓
@狄公 (QA)          →  独立验证，三张审查清单
    ↓ 打回(≤3次) → 回到 @鲁班
    ↓ 3次打回 → @司南 仲裁
    ↓ 通过
@墨子 (DevOps)      →  构建 → 测试 → 部署
    ↓
@云笈 (原型切换)    →  可叠加到任意角色，切换行为原型
```

## 五行为原型

> **运周天，深入木，留存真，弹绳墨，准定盘。**

五行为原型是鲁班学派的核心行为方法论。命名取自**天地法器**——道家修行哲学与工匠器物的交汇。每个原型是一套完整的正交方法论，可独立使用，也可通过 `@云笈` 灵活叠加到任意角色。

| 原型 | 功法 | 口诀 | 天象 | 方法论 | 适用场景 |
|------|------|------|------|--------|---------|
| 周天 | 周天功 | 运周天 | 风起云涌 | Plan → Execute → Verify → Retrospect | 规划、部署、交付 |
| 入木 | 入木诀 | 深入木 | 劫云至 | 5-Why → Evidence → Prevent | Debug、排错、审查 |
| 存真 | 存真式 | 留存真 | 云开雾散 | Question → Delete → Simplify → Build | 编码、重构、优化 |
| 绳墨 | 绳墨令 | 弹绳墨 | 雷云压顶 | Keeper Test → Replace/Approve | 代码审查、质量门禁 |
| 定盘 | 定盘术 | 准定盘 | 风停云住 | Measure → A/B → Evidence | 性能优化、部署验证 |

### 后缀释义

功、诀、式、令、术——各取其性，不混用。

| 后缀 | 本义 | 对应原型 | 为何是它 |
|------|------|---------|---------|
| **功** | 功法，持续修炼 | 周天 | 循环不息，日日运功 |
| **诀** | 口诀，关键时刻激活 | 入木 | 念诀即入深层，遇疑难方激活 |
| **式** | 招式，有起手有收招 | 存真 | 从质疑起手，到构建收招 |
| **令** | 法令，不可违逆 | 绳墨 | 越线即斩，无例外 |
| **术** | 术法，精准度量 | 定盘 | 校准基准，称量斤两 |

### 天象谱系

五条真言的天象构成一组完整的气象谱——从风起到云住，对应从启动到收功的全周期：

```
风起云涌 → 劫云至 → 云开雾散 → 雷云压顶 → 风停云住
  周天       入木       存真       绳墨       定盘
  启动       穿透       澄明       裁决       归静
```

## 核心标识

| 标识 | 含义 |
|------|------|
| `[准绳]` | 诊断先行——弹了墨线再动手 |
| `[匠心 🔨]` | 超出要求的工匠精神 |
| `[榫卯]` | 任务交付——完美契合，无需返工 |
| `三准绳` | 闭环 / 事实驱动 / 穷尽 |
| `C0-C4` | 匠心入微——五个精进层次 |

## 安装

### 分层选择

| 层级 | 安装内容 | 适用场景 |
|------|---------|---------|
| **最小** | 1 个角色（如只装鲁班） | 个人开发者，只需编码助手 |
| **标准** | 4 角色（司南+鲁班+狄公+墨子） | 团队协作，完整流水线 |
| **完整** | 全部 6 个（+云笈+绳墨） | 需要原型切换+深入行为体系 |

### Claude Code

**插件安装（推荐，装完即用）：**

```bash
claude plugin marketplace add luban-school/luban-school
claude plugin install luban-school@luban-school
```

安装后直接使用斜杠命令：`/luban:司南` `/luban:鲁班` `/luban:狄公` `/luban:墨子` `/luban:云笈`。无需手动配置 CLAUDE.md。

**手动项目级安装：**

```bash
mkdir -p .claude/skills
cp -r path/to/luban-school/skills/* .claude/skills/
mkdir -p .claude/commands
cp -r path/to/luban-school/commands/* .claude/commands/
```

手动安装后，在项目 CLAUDE.md 中添加触发规则（可选，用于 @mention 触发）：

```markdown
## 角色技能强制触发

当用户消息包含以下 @提及，MUST 调用 Skill 工具。

| 触发词 | Skill 名 | 角色 |
|--------|---------|------|
| @司南 | sinan | Tech Lead |
| @鲁班 | luban | Engineer |
| @狄公 | digong | QA |
| @墨子 | mozi | DevOps |
| @云笈 | yunji | Archetype |
```

### Vercel Skills CLI

```bash
# 标准安装（4 角色）
npx skills add luban-school/luban-school --skill sinan
npx skills add luban-school/luban-school --skill luban
npx skills add luban-school/luban-school --skill digong
npx skills add luban-school/luban-school --skill mozi

# 可选：原型切换 + 行为引擎
npx skills add luban-school/luban-school --skill yunji
npx skills add luban-school/luban-school --skill shengmo
```

### 标准 SKILL.md 平台

Codex CLI / CodeBuddy / OpenCode / OpenClaw / Google Antigravity 均支持标准 SKILL.md 格式。将 `skills/` 目录复制到对应平台即可：

| 平台 | 安装目录 | 触发方式 |
|------|---------|---------|
| Codex CLI | `~/.codex/skills/` | description 自动匹配 + `$角色名` |
| CodeBuddy | `~/.codebuddy/skills/` | plugin 机制 |
| OpenCode | `~/.config/opencode/skills/` | 标准 SKILL.md |
| OpenClaw | `~/.openclaw/skills/` | ClawHub |
| Antigravity | `~/.gemini/antigravity/skills/` | 标准 SKILL.md |

### 非标平台

| 平台 | 配置文件 | 安装方式 |
|------|---------|---------|
| Cursor | `cursor/rules/luban-school.mdc` | 复制到 `.cursor/rules/`，AI 语义自动匹配 |
| Kiro | `kiro/steering/luban-school.md` | 复制到 `.kiro/steering/`，始终生效 |
| VSCode Copilot | `vscode/copilot-instructions.md` | 复制到 `.github/copilot-instructions.md` |

---

## 使用方法

在 Claude Code 中（安装插件后），斜杠命令或 @mention 均可触发：

```
# 斜杠命令（插件安装后立即可用，推荐）
/luban:司南 我需要一个用户登录功能
/luban:鲁班 开始执行
/luban:狄公 审查
/luban:墨子 deploy
/luban:yunji 入木 @鲁班

# @mention（手动安装需配置 CLAUDE.md 触发规则）
@司南 我需要一个用户登录功能
@鲁班 开始执行
@狄公 审查
@墨子 deploy
@云笈(入木) @鲁班
```

手动安装时，确保项目 CLAUDE.md 包含触发规则（见上方模板）。

---

## 技能文件

| Skill | 角色 | 默认原型 |
|-------|------|---------|
| `shengmo` | 绳墨 — 行为引擎 | — |
| `sinan` | 司南 — Tech Lead | 周天 |
| `luban` | 鲁班 — 执行工程师 | 存真 |
| `digong` | 狄公 — 质量守门人 | 入木 |
| `mozi` | 墨子 — 部署运维 | 周天 |
| `yunji` | 云笈 — 原型切换器 | — |

## License

MIT。详见 [LICENSE](LICENSE)。

---

## Credits

鲁班学派 — 契约驱动的工匠精神。
