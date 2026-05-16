# 巡天指令簿

> 巡天仪向周天仪/司南/仙人发送干预指令的共享文件。
> 周天仪启动后应读取本文件，响应 target=zhoutian 的 pending 指令。
> 归档区是 escalation chain 的数据源，每次 cron 触发时更新。

---

<!-- 无待处理指令 —— 正常流无障碍 -->

## 历史指令归档

<!-- 格式: - INT-NNN（timestamp）· target=X · type=Y → **状态**（说明） -->
<!-- 超过 50 条 INT 时，最旧 30 条自动归档到 intervention-archive.md -->

<!-- BATCH-B 初始化，无历史指令 -->
