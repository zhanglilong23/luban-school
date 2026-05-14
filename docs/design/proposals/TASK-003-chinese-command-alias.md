# TASK-003 · 中文斜杠命令调用问题 — 技术方案

**契约来源**: 司南 2026-05-14 残留物治理 Sprint
**类型**: need-design（等老板拍板）
**鲁班推荐**: **方案 C**（混合策略：英文文件名 + 显式映射）
**影响范围**: `.claude-plugin/plugin.json` / `hooks/hooks.json` / `commands/` / `README.md` / `README.zh-CN.md` / `commands/*.md` 标题

---

## 一、问题陈述

Claude Code 斜杠命令名 = `commands/*.md` 文件名（去 `.md` 后缀）。当前文件名全是英文：

```
commands/wendao.md    →  /luban:wendao
commands/sinan.md     →  /luban:sinan
commands/luban.md     →  /luban:luban
commands/digong.md    →  /luban:digong
commands/mozi.md      →  /luban:mozi
commands/setup.md     →  /luban:setup
```

但项目内多处文档**推广中文命令**（用户实际无法调用）：

| 文件 | 行号 | 内容 |
|------|------|------|
| `plugin.json` | L4 | `斜杠命令 /luban:问道 /luban:司南 /luban:鲁班 /luban:狄公 /luban:墨子 激活角色` |
| `hooks/hooks.json` | L10 | `[鲁班学派] 斜杠命令: /luban:问道 /luban:司南 ...` |
| `README.md` | L140-144 | `/luban:问道 我觉得功能不够智能` 等示例 |
| `README.zh-CN.md` | L127, L195-204 | `/luban:问道` `/luban:司南` 安装提示与示例 |
| `commands/wendao.md` | L6 | 标题 `# /luban:问道 — 需求求道者`（与速查表 `/luban:wendao` 自我矛盾） |

**狄公审查结论**: doc-as-code 闭环失败 —— 文档承诺的命令格式与实际可调用形式不一致，用户复制粘贴必踩坑。

---

## 二、Claude Code 命令机制（事实）

Claude Code 斜杠命令的命名约束：
- 命令名 = 文件名 stem（去 `.md`）
- frontmatter 无官方 `alias` 字段
- 文件名 ASCII 安全性更高（Windows / git / shell 跨平台无 encoding 风险）
- 中文文件名理论可用，但增加跨平台风险面

---

## 三、备选方案对比

### 方案 A — commands 加中文文件名（冗余镜像）

新增 6 个中文文件名（`问道.md` `司南.md` `鲁班.md` `狄公.md` `墨子.md` + 现有英文保留），内容简短转发到 Skill。

**利**:
- 用户中文命令直接可用
- 文档无需改动

**弊**:
- 文件冗余翻倍（6 → 12）
- 中文文件名 git / Windows / shell 跨平台兼容性风险
- 删改双倍工作（角色重命名要改两份）
- plugin 包大小翻倍
- 维护成本长期上升

**工时**: ≤1h
**风险**: 中（跨平台兼容性）

### 方案 B — 文档全改英文（doc-as-code 强对齐）

`README.md` / `README.zh-CN.md` / `hooks.json` / `plugin.json` / `commands/*.md` 标题全改为 `/luban:wendao` 等英文形式；`@提及` 保留中文 `@问道 @司南`。

**利**:
- doc-as-code 完全闭环 —— 文档承诺的就是可调用的
- 零冗余，最稳定
- 跨平台无风险

**弊**:
- 中文品牌感被英文命令稀释
- 已习惯中文形式的早期用户有迁移成本
- 中英两种调用形式（命令英文、@提及中文）需要心智切换

**工时**: ≤1h
**风险**: 低

### 方案 C — 混合策略：英文文件名 + 显式映射（鲁班推荐）

保留单一英文文件名为 Claude Code 调用入口。在以下位置**显式标注中英映射**：
- `hooks/hooks.json` 启动横幅加映射提示
- `README.md` / `README.zh-CN.md` "使用方法"章节加映射表
- `commands/*.md` 文件标题统一为英文 `/luban:wendao` 形式（修复自我矛盾）
- @提及 保留中文（CLAUDE.md 不动）

映射表样式（写进 README "使用方法"）：

```markdown
| 角色 | 斜杠命令 | @提及 |
|------|---------|-------|
| 问道 | `/luban:wendao` | `@问道` |
| 司南 | `/luban:sinan`  | `@司南` |
| 鲁班 | `/luban:luban`  | `@鲁班` |
| 狄公 | `/luban:digong` | `@狄公` |
| 墨子 | `/luban:mozi`   | `@墨子` |
```

**利**:
- doc-as-code 闭环 —— 文档承诺映射，映射就是真实可调用的
- 兼顾中文品牌感（@提及 中文）与可用性（命令英文）
- 零文件冗余
- 跨平台稳定

**弊**:
- 用户初次见到映射需要看一眼（一次性认知成本）

**工时**: ≤1h
**风险**: 低

---

## 四、鲁班推荐与理由

**[准绳] 推荐方案 C**，理由：

1. **CLAUDE.md L9-17 触发规则表里 @提及 本来就是中文** —— 用户日常自然语言交互用 `@问道 @司南` 完全没问题，中文品牌感由 @提及 承担
2. **斜杠命令是 Claude Code 工程形式**，应遵循其文件名约定（ASCII 英文最稳）
3. **显式映射 = 一次性成本，零长期成本** —— 用户看一次表就记住了
4. **方案 A 维护成本随角色增减线性上升** —— 角色重命名要改两个文件、两套描述，违背 DRY
5. **方案 B 损失中文品牌感太干净** —— 鲁班学派的中文修仙叙事是核心卖点

---

## 五、待老板拍板

请老板从 A / B / C 中选一个。选定后，鲁班按选定方案立即执行落盘改动（≤1h）。
