# 任务拆解契约 — AUTO-PIPELINE-001

**日期**: 2026-05-14
**状态**: 已确认 / 执行中

## WHY

保留七签五角色架构不变，新增一个编排 Skill 实现下游自动化自治。

## 任务

| ID | 描述 | 工时 | 风险 | 模式 |
|:---|:---|:---|:---|:---|
| TASK-001 | 新增自动化编排 Skill | ≤4h | 中 | [direct-coding] |

- **文件**: `skills/auto/SKILL.md`
- **触发**: `/luban:auto TASK-xxx`
- **内部**: spawn 鲁班 → 检测[榫卯] → spawn 狄公 → 检测放行单 → spawn 墨子
- **重试**: 每角色异常最多重试 3 次，第 4 次暂停输出状态报告等用户
- **留痕**: 输出结构化执行日志（时间/角色/动作/结果）

## DONE

- [x] 新增 `skills/auto/SKILL.md`，支持 `/luban:auto TASK-xxx` 调用
- [x] 内部编排：按顺序 spawn 鲁班→狄公→墨子，传递契约上下文
- [x] 重试逻辑：每角色异常自动重试 ≤3 次，第4次暂停等用户决策
- [x] 状态检测：检测 [榫卯] 标记、《质量放行单》作为阶段门禁
- [x] 留痕：输出结构化执行报告 `[AUTO-REPORT] task/role/action/result/retries`
- [x] 契约落盘到 `docs/luban-school/contracts/2026-05-14-auto-pipeline.md`

## DON'T

- 禁止修改: `skills/wendao/SKILL.md`, `skills/sinan/SKILL.md`, `skills/luban/SKILL.md`, `skills/digong/SKILL.md`, `skills/mozi/SKILL.md`
- 禁止修改: `CLAUDE.md`, `orchestration-patterns.md`
- 禁止: 删除现有七签内容、引入外部依赖框架

## 确认记录

- 需求真言: @问道 炼成
- 契约拆解: @司南 输出
- 执行编码: @鲁班 交付
- 待审查: @狄公
