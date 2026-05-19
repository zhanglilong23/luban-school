# ROLE-CHANGE-CHECKLIST · 角色增删改 SOP

**作用**: 防止角色增删改时漏改触点。yunji/shengmo 废除遗留就是反面教材。
**强制范围**: 任何新增、重命名、废除角色的 PR 必须在描述里贴自检模板（见第六节）。

---

## 一、问题陈述

云笈（yunji）和神魔（shengmo）角色废除时，留下了 **多处残留触点未同步更新**：

- 旧契约 DONE 第 5/14 行还引用 `yunji/SKILL.md`
- `docs/orchestration-patterns.md` 还在用 `@云笈(入木) @鲁班` 旧语法
- `.claude/settings.local.json` 还有 `shengmo` 的 cp 权限
- `hooks/hooks.json` 启动横幅还含 `/luban:云笈 @云笈`

**根因**: 角色增删改影响的文件分散在 12+ 处，没有 checklist，全靠脑子记，必漏。

---

## 二、ROLE-CHANGE-CHECKLIST（正式 SOP）

每次新增、重命名、废除角色时，按以下清单逐项确认。**未走完不算闭环**。

### A. 角色身份触点（必须）

| # | 文件 | 改动点 | 适用情景 |
|---|------|--------|---------|
| 1 | `.claude-plugin/plugin.json` | `description` 字段角色列表 | 新增/重命名/废除 |
| 3 | `skills/<角色>/SKILL.md` | 新建/重命名/删除 Skill 文件 | 新增/重命名/废除 |
| 4 | `hooks/hooks.json` | SessionStart 启动横幅角色列表 | 新增/重命名/废除 |
| 5 | `CLAUDE.md` | 角色技能强制触发表 | 新增/重命名/废除 |
| 6 | `CONTEXT.md` | 角色体系定义、术语 _Avoid_ 表 | 新增/重命名/废除 |
| 7 | `README.md` | 架构图、技能文件表、使用示例 | 新增/重命名/废除 |
| 8 | `README.zh-CN.md` | 架构图、技能文件表、使用示例 | 新增/重命名/废除 |

### B. 协作/编排触点（必须）

| # | 文件 | 改动点 | 适用情景 |
|---|------|--------|---------|
| 9 | `docs/orchestration-patterns.md` | 模式三协作链、反模式表 | 新增/重命名/废除 |
| 10 | 每个 `skills/<角色>/SKILL.md` | 修仙位格示意图（角色链 + 默认签关系） | 新增/重命名/废除 |

### C. 三方平台镜像（必须）

| # | 文件 | 改动点 | 适用情景 |
|---|------|--------|---------|
| 11 | `cursor/rules/luban-school.mdc` | Cursor 规则中角色列表 | 新增/重命名/废除 |
| 12 | `kiro/steering/luban-school.md` | Kiro Steering 中角色列表 | 新增/重命名/废除 |
| 13 | `vscode/copilot-instructions.md` | VSCode Copilot 指令中角色列表 | 新增/重命名/废除 |

### D. 权限/版本/命令别名（视情况）

| # | 文件 | 改动点 | 适用情景 |
|---|------|--------|---------|
| 14 | `.claude/settings.local.json` | `Skill(<plugin>:<角色>)` 白名单；`Bash(cp ...)` 残留权限清理 | 新增/废除 |
| 15 | `.claude-plugin/plugin.json` | `version` 字段（重大变更建议升 minor） | 任何变更 |
| 16 | `.claude-plugin/marketplace.json`（若存在） | `metadata.version` 字段 | 任何变更 |

### E. 历史契约/设计稿（必须）

| # | 文件 | 改动点 | 适用情景 |
|---|------|--------|---------|
| 17 | `docs/design/archive/` 下的旧契约/设计稿 | 若引用废除角色，加 ARCHIVED banner | 废除 |
| 18 | `docs/luban-school/contracts/` 下的活跃契约 | 是否有引用废除角色的 DONE 项需要废止 | 废除 |

### F. 闭环验证（必须）

| # | 命令 | 验证内容 |
|---|------|---------|
| 19 | `grep -rE "<废除角色名>" --include="*.md" --include="*.json" --include="*.mdc"` | 确认仅在归档目录或合法历史语境命中 |
| 20 | `grep -rE "<新角色名>" --include="*.md" --include="*.json" --include="*.mdc"` | 确认 A/B/C 项中所有应改之处都已改 |

---

## 三、新增角色专属步骤

新增角色除以上清单外，额外：

1. **职责边界 4 件套**：在 CONTEXT.md 给 _Avoid_ 列表；在 SKILL.md 给"职责边界 / 输入 / 输出 / 绝不"四段
2. **协作链插入位**：明确新角色在 `@问道 → @司南 → @鲁班 → @狄公 → @墨子` 哪两个之间
3. **修仙位格命名**：若引入新位格（如"飞升期"），需在 CLAUDE.md + 所有 SKILL.md 修仙位格示意图同步
4. **七签主修匹配**：声明新角色的默认签（七签现有 7 个，必要时考虑扩展第八签——这是大事，须经司南架构仲裁）

---

## 四、重命名专属步骤

1. **Git mv 而非 rm + add**：保留历史
2. **commit message 含旧名→新名映射**：未来 grep 旧名能找到 commit
3. **README/CHANGELOG 加迁移说明**：用户从旧版升级时能看到

---

## 五、废除角色专属步骤

1. **不删，归档**：`mv` 到 `docs/design/archive/`，加 ARCHIVED banner（含归档时间、原因、替代角色）
2. **历史契约加 ARCHIVED banner**：引用废除角色的旧契约不删，加 banner（参考 `docs/design/archive/2026-05-13-stable-catchphrases-ARCHIVED.md`）
3. **CHANGELOG 写明废除原因**：避免日后有人疑问"为什么没有 yunji 了"

---

## 六、Checklist 自检模板（粘贴到 PR 描述）

```markdown
## ROLE-CHANGE-CHECKLIST 自检

- [ ] A. 角色身份触点 1-8 全部已检查
- [ ] B. 协作/编排触点 9-10 全部已检查
- [ ] C. 三方平台镜像 11-13 全部已检查
- [ ] D. 权限/版本 14-16 已视情况处理
- [ ] E. 历史契约/设计稿 17-18 已视情况归档
- [ ] F. 闭环验证 19-20 grep 命令已跑，输出贴在下方

**grep 验证输出**:
```
{贴 grep 命令输出}
```
```

---

## 七、变更历史

- 2026-05-14 落盘正式 SOP（来源：司南 2026-05-14 残留物治理 Sprint TASK-009）。yunji 废除遗留事件促成本清单。
