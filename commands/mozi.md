---
description: "部署运维 — 构建/测试/部署流水线，Agent 资源生命周期管理。"
argument-hint: "[deploy|agent]"
---

# /luban:墨子 — DevOps

加载 Skill("mozi") 激活墨子角色。

两种模式：
- `deploy` — 狄公放行后触发：构建 → 测试 → 部署 → 健康检查
- `agent` — 任务结束后触发：停止 Agent、清理 worktree、释放端口
