# 鲁班学派

<p align="center">
  <b>契约驱动的 AI Agent 角色分工协作体系</b>
</p>

五角色、七签功法、一份契约。修仙体系：求道→筑基→结丹→化神→渡劫。云笈是天书（功法总纲），不是角色。问道从混沌中炼需求真言、司南拆需求定契约、鲁班编码闭环交付、狄公独立质量审查、墨子部署运维。每个角色自包含完整七签定义，独立安装即可使用。

**三重能力：**

1. **角色分工协作** — 五角色边界清晰，契约驱动交付
2. **七签功法体系** — 七个正交方法论，按场景切换，内置每个角色
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
@问道 (需求求道者)  →  从混沌中炼出需求真言
    ↓
@司南 (Tech Lead)   →  输出《任务拆解契约》(TASK-ID + DONE + DON'T)
    ↓
@鲁班 (Engineer)    →  按契约编码，交付 [榫卯]
    ↓
@狄公 (QA)          →  独立验证，三张审查清单
    ↓ 打回(≤3次) → 回到 @鲁班
    ↓ 3次打回 → @司南 仲裁
    ↓ 通过
@墨子 (DevOps)      →  构建 → 测试 → 部署
```

### 修仙位格与七签

```
求道（问道·问道签）→ 筑基（司南·周天签）→ 结丹（鲁班·存真签）→ 化神（狄公·绳墨签）→ 渡劫（墨子·定盘签）
                                                                      ↑
                                                          入木签（公共 Debug）+ 契约签（宪法）贯穿始终
```

从混沌到生产，恰好是一个修士从求道到渡劫的完整路径。七签是修行之术，契约是万法归一的根本准则。每个角色 SKILL.md 已内嵌完整七签定义，独立安装即可使用全部功法。

## 七签功法

> **炼真言 / 运周天 / 深入木 / 留存真 / 弹绳墨 / 准定盘 / 定契约**

七签是鲁班学派的核心功法体系。命名取自道教经典《云笈七签》。每签是一套完整的正交方法论，通过 `@角色 +签名` 修习。

| 签序 | 签名 | 法门 | 口诀 | 性质 | 主修 | 适用场景 |
|------|------|------|------|------|------|---------|
| 第一签 | 问道签 | 问道心法 | 炼真言 | 心法 | 问道 | 需求求道 |
| 第二签 | 周天签 | 周天功 | 运周天 | 功法 | 司南 | 规划、部署、交付 |
| 第三签 | 入木签 | 入木诀 | 深入木 | 诀法 | 公共 Debug 签 | Debug、排错、审查 |
| 第四签 | 存真签 | 存真式 | 留存真 | 式法 | 鲁班 | 编码、重构、优化 |
| 第五签 | 绳墨签 | 绳墨令 | 弹绳墨 | 令法 | 狄公 | 代码审查、质量门禁 |
| 第六签 | 定盘签 | 定盘术 | 准定盘 | 术法 | 墨子 | 性能优化、部署验证 |
| 第七签 | 契约签 | 契约法 | 定契约 | 大法 | 全员（宪法） | 无契约，不行动 |

**修习语法**: `@角色 +签名` / `@角色 -签名` / `@角色 +签1 +签2`

入木签为公共 Debug 签，任意角色皆可修习。契约签为全员宪法，永远在线。

### 后缀释义

心法、功法、诀法、式法、令法、术法、大法——各取其性，不混用。

| 后缀 | 本义 | 对应签 | 为何是它 |
|------|------|--------|---------|
| **心法** | 内心修炼 | 问道签 | 求道在心，不在术 |
| **功法** | 功法，持续修炼 | 周天签 | 循环不息，日日运功 |
| **诀法** | 口诀，关键时刻激活 | 入木签 | 念诀即入深层，遇疑难方激活 |
| **式法** | 招式，有起手有收招 | 存真签 | 从质疑起手，到构建收招 |
| **令法** | 法令，不可违逆 | 绳墨签 | 越线即斩，无例外 |
| **术法** | 术法，精准度量 | 定盘签 | 校准基准，称量斤两 |
| **大法** | 根本大法 | 契约签 | 万法归一，契约为纲 |

### 七签口诀

```
炼真言 / 运周天 / 深入木 / 留存真 / 弹绳墨 / 准定盘 / 定契约
  一       二       三       四       五       六       七
 心法     功法     诀法     式法     令法     术法     大法
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
| **标准** | 5 角色（问道+司南+鲁班+狄公+墨子） | 团队协作，完整流水线 |

每个角色自包含完整七签定义，独立安装即可使用全部功法。

### Claude Code

**插件安装（推荐，装完即用）：**

```bash
claude plugin marketplace add luban-school/luban-school
claude plugin install luban-school@luban-school
```

安装后直接使用斜杠命令：`/luban:wendao` `/luban:sinan` `/luban:luban` `/luban:digong` `/luban:mozi`。中英映射详见下方"使用方法"。无需手动配置 CLAUDE.md。

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
| @问道 | wendao | 需求求道者 |
| @司南 | sinan | Tech Lead |
| @鲁班 | luban | Engineer |
| @狄公 | digong | QA |
| @墨子 | mozi | DevOps |
```

### Vercel Skills CLI

```bash
# 标准安装（5 角色）
npx skills add luban-school/luban-school --skill wendao
npx skills add luban-school/luban-school --skill sinan
npx skills add luban-school/luban-school --skill luban
npx skills add luban-school/luban-school --skill digong
npx skills add luban-school/luban-school --skill mozi
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

### 命令 ↔ 角色映射

| 角色 | 斜杠命令 | @提及 |
|------|---------|-------|
| 问道 | `/luban:wendao` | `@问道` |
| 司南 | `/luban:sinan` | `@司南` |
| 鲁班 | `/luban:luban` | `@鲁班` |
| 狄公 | `/luban:digong` | `@狄公` |
| 墨子 | `/luban:mozi` | `@墨子` |

斜杠命令采用 ASCII 文件名确保跨平台稳定；@提及保留中文角色名以符合自然语言习惯。

```
# 斜杠命令（插件安装后立即可用，推荐）
/luban:wendao 我觉得功能不够智能
/luban:sinan 我需要一个用户登录功能
/luban:luban 开始执行
/luban:digong 审查
/luban:mozi deploy

# @mention（手动安装需配置 CLAUDE.md 触发规则）
@问道 我觉得功能不够智能
@司南 我需要一个用户登录功能
@鲁班 开始执行
@狄公 审查
@墨子 deploy
@鲁班 +入木          # 鲁班修习入木签
@狄公 +存真          # 狄公修习存真签
```

手动安装时，确保项目 CLAUDE.md 包含触发规则（见上方模板）。

---

## 技能文件

| Skill | 角色 | 默认签 | 修仙位格 |
|-------|------|--------|---------|
| `wendao` | 问道 — 需求求道者 | 问道签 | 求道期 |
| `sinan` | 司南 — Tech Lead | 周天签 | 筑基期 |
| `luban` | 鲁班 — 执行工程师 | 存真签 | 结丹期 |
| `digong` | 狄公 — 质量守门人 | 绳墨签 | 化神期 |
| `mozi` | 墨子 — 部署运维 | 定盘签 | 渡劫期 |

每个技能文件自包含：行为基线 + 七签速查 + 全部签文 + 修习语法 + 专属规程。

## License

MIT。详见 [LICENSE](LICENSE)。

---

## Credits

鲁班学派 — 契约驱动的工匠精神。
