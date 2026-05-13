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
