> ⚠️ **ARCHIVED · 历史契约** — 本契约已归档，**不代表当前完成状态**。
> 归档时间: 2026-05-14 | 归档原因:
> - DONE 第 5 项 `yunji/SKILL.md` 已失效（yunji skill 已废除）
> - 实际 commit `6b609fa` 信息"口头禅升级为 HARD-GATE"与代码不符（口头禅段未用 `<HARD-GATE>` 标签包裹），契约伪完成
> - 替代契约见 v1.3.0 残留物治理 Sprint（司南 2026-05-14 重立）

---

## 任务拆解契约

**WHY**: 口头禅从"描述性段落"升级为 HARD-GATE 精确输出指令，防止 LLM 自由发挥导致口头禅变幻或消失。

| ID | 描述 | 工时 | 风险 | 模式 |
|----|------|------|------|------|
| TASK-001 | 五角色 SKILL.md 口头禅 HARD-GATE 化 | ≤2h | 低 | [direct-coding] |

**DONE**:
- [ ] sinan/SKILL.md: 口头禅改为 `<HARD-GATE>` 精确输出，使用用户认可的 "司南已就位，运周天，观全局。"
- [ ] luban/SKILL.md: 口头禅改为 `<HARD-GATE>` 精确输出
- [ ] digong/SKILL.md: 口头禅改为 `<HARD-GATE>` 精确输出
- [ ] mozi/SKILL.md: 口头禅改为 `<HARD-GATE>` 精确输出
- [ ] yunji/SKILL.md: 口头禅改为 `<HARD-GATE>` 精确输出（五原型动态选择保留）
- [ ] 五个文件 verbal diff 确认无遗漏

**DON'T**:
- 文件白名单: skills/sinan/SKILL.md, skills/luban/SKILL.md, skills/digong/SKILL.md, skills/mozi/SKILL.md, skills/yunji/SKILL.md
- 禁止: 改动口头禅文本以外的内容（行为基线、角色规程、常见借口、行为边界均不动）
- 禁止: 引入新依赖或新文件
