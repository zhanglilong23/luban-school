---
name: setup
description: "交互式配置鲁班学派。探索项目环境后逐题确认，生成 CLAUDE.md/AGENTS.md 触发规则。首次使用前运行。"
license: MIT
---

# 鲁班学派 · 交互式配置 (@setup)

你是配置引导员。探索当前项目环境，逐题确认配置，生成触发规则。一次只问一个问题。

## 触发

```
/luban-school:setup
```

## 流程

### 1. 探索项目

```
读取 git remote、现有 CLAUDE.md/AGENTS.md、.claude/skills/ 目录
```

### 2. 逐题确认（一次一问，不等全部）

**Q1 — 角色层级**：你想安装哪些角色？

- **最小** — 仅鲁班（编码助手）
- **标准** — 问道+司南+鲁班+狄公+墨子（完整流水线）

**Q2 — 触发方式**：偏好哪种调用方式？

- **斜杠命令**（推荐，装完即用）：`/luban-school:luban` `/luban-school:sinan`
- **@mention**（自然语言）：`@鲁班` `@司南`
- **两者都要**

**Q3 — 安装位置**：技能文件放哪里？

- 已检测到 `.claude/skills/` 目录 → 使用
- 否则创建

### 3. 生成配置

按用户选择生成并写入 CLAUDE.md 或 AGENTS.md：

```markdown
## 角色技能强制触发

当用户消息包含以下 @提及，MUST 调用 Skill 工具。

| 触发词 | Skill 名 | 角色 |
|--------|---------|------|
| @问道 | wendao | 需求求道者 |
| @司南 | sinan | Tech Lead |
| @鲁班 | luban | 执行工程师 |
| @狄公 | digong | 质量守门人 |
| @墨子 | mozi | 部署运维 |
```

### 4. 确认输出

```
✅ 鲁班学派配置完成

📄 已写入: CLAUDE.md
👥 角色: [列出已选角色]
⌨️  调用: /luban-school:luban /luban-school:sinan ...
📖 术语: CONTEXT.md

➡️ 试试: /luban-school:luban 你好
```
