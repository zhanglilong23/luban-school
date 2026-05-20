# TASK-010 · HARD-GATE 设计范围定性 — 技术方案

**契约来源**: 司南 2026-05-14 残留物治理 Sprint
**类型**: need-design → **已决议 HG-2，已落地**
**鲁班推荐**: **方向 HG-1**（仅门禁段使用 HARD-GATE，CLAUDE.md 加一句明示）
**影响范围**: `CLAUDE.md`（加定义段）+ 视情况 `skills/wendao/SKILL.md` `skills/mozi/SKILL.md`

---

## 一、问题陈述

### 现状（grep 实测）

```
skills/digong/SKILL.md  L144-158   绳墨令门禁段（独立验证三张清单）
skills/luban/SKILL.md   L120-129   鲁班门禁段（无契约必拒）
skills/sinan/SKILL.md   L144-148   司南门禁段（契约三要素必备）
skills/wendao/SKILL.md  ❌         无 HARD-GATE 标签
skills/mozi/SKILL.md    ❌         无 HARD-GATE 标签
```

**三个角色用，两个角色不用** —— 范围不一致。

### 历史背景

归档契约 `docs/design/archive/2026-05-13-stable-catchphrases-ARCHIVED.md` 的原始目标是"**五角色统一**口头禅 HARD-GATE 化"，但：
- 该契约 DONE 第 5 项是 `yunji/SKILL.md`（已废除角色）
- 实际 commit `6b609fa` 信息"口头禅升级为 HARD-GATE"与代码不符（口头禅段未用 HARD-GATE 标签包裹）
- 契约伪完成 —— 已归档

**归档不等于决议** —— "HARD-GATE 该不该用于五角色"这个设计问题还悬着。

### CLAUDE.md 现状

`CLAUDE.md` 提到了核心标识表（L43-51），但**没有定义 HARD-GATE**。HARD-GATE 是 SKILL.md 里 ad-hoc 使用的语法，缺少宪法级定调。

---

## 二、备选方向

### 方向 HG-1 — 仅门禁段使用 HARD-GATE（鲁班推荐）

**定性**: HARD-GATE 是"**不可妥协的输出指令**"标签，仅用于真正需要硬约束格式的段落（门禁/契约三要素/审查清单等）。非门禁段（如口头禅、追问、性格描写）**不强制**用 HARD-GATE，保留 LLM 表达灵活性。

**改动**:
- `CLAUDE.md` 新增"HARD-GATE 定义"段，明示范围 = 门禁/契约/审查清单
- `skills/wendao/SKILL.md` `skills/mozi/SKILL.md` **不改** —— wendao 是追问开放性输出、mozi 是部署日志，本来就不需要 HARD-GATE

**利**:
- 与现状一致，改动最小
- 保留 wendao 追问、mozi 健康检查的输出多样性（这两个角色的输出本来就该是动态的）
- HARD-GATE 语义清晰 —— "格式硬约束" vs "性格灵活表达"分离

**弊**:
- 五角色看起来"不对称"（三角色有 HARD-GATE 段，两角色没有）—— 但其实是按需使用而非缺失

**工时**: ≤30min（只改 CLAUDE.md）
**风险**: 低

### 方向 HG-2 — 五角色统一 HARD-GATE 化

**定性**: HARD-GATE 用于所有角色的"必须原样输出"段落（包括口头禅）。wendao 和 mozi 也补上 HARD-GATE 口头禅段。

**改动**:
- `CLAUDE.md` 新增"HARD-GATE 定义"段，明示范围 = 所有"原样输出"段落
- `skills/wendao/SKILL.md` 口头禅段加 `<HARD-GATE>` 包裹
- `skills/mozi/SKILL.md` 口头禅段加 `<HARD-GATE>` 包裹

**利**:
- 五角色风格统一，无对称性问题
- 口头禅锁死，防 LLM 自由发挥（这是归档契约的原始 WHY）

**弊**:
- wendao 口头禅是"温和固执的追问"，强行原样输出会丢失追问的灵活性
- mozi 口头禅 "🛡️ 墨子接管。环境、构建、清理，我只动真格。" 已经够硬，加 HARD-GATE 是过度工程
- 改动 3 个文件（CLAUDE.md + 2 个 SKILL.md），工时翻倍

**工时**: ≤1h
**风险**: 中（可能损失 wendao 追问的开放性）

---

## 三、鲁班推荐与理由

**[准绳] 推荐方向 HG-1**，理由：

1. **HARD-GATE 的本意是"格式不可妥协"** —— 门禁/契约/审查清单确实有"必须原样输出"的格式要求；口头禅虽然也希望稳定，但允许 LLM 在风格内微调反而更自然
2. **wendao 是开放式追问角色** —— 强加 HARD-GATE 等于把追问写死，与"求道"的灵活性矛盾
3. **mozi 是部署日志角色** —— 输出形式天然多样（健康检查、构建日志、回滚状态），HARD-GATE 不适用
4. **归档契约因 yunji 失效已废止** —— "五角色统一"这个目标本身就是历史包袱
5. **CLAUDE.md 加一句明示就够** —— 不需要改 SKILL.md，工时最小

### CLAUDE.md 拟新增段（方向 HG-1）

在 CLAUDE.md "核心标识"表之后，新增：

```markdown
## HARD-GATE 设计范围

`<HARD-GATE>...</HARD-GATE>` 标签用于"**不可妥协的输出指令**"——LLM 必须原样输出标签内文本/结构，不得改写、省略或自由发挥。

**使用范围**:
- 司南契约三要素门禁（TASK-ID + DONE + DON'T 缺一不可）
- 狄公审查清单（三张清单 + 放行单/打回单格式）
- 鲁班无契约拒绝执行门禁

**不使用**: 口头禅、性格描写、开放式追问 —— 这些保留 LLM 表达灵活性。
```

---

## 四、决议

**选定 HG-2**（五角色统一 HARD-GATE 化）。

- CLAUDE.md 已新增 HARD-GATE 定义段，明示"五角色统一使用范围（HG-2 定调）"
- wendao/mozi SKILL.md 口头禅已加 `<HARD-GATE>` 包裹
- 不使用范围：开放式追问、性格描写、签文激活语

**归档原因**：设计决策已落地执行，CLAUDE.md + 五角色 SKILL.md 均已更新，提案使命完成。
