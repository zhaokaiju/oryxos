# CLI Contract

- `oryxos init`: 创建 `.oryxos/agents`、`skills`、`memory`、`sessions`、`logs` 等工作区目录；重复执行幂等。
- `oryxos chat --profile <name>`: 通过指定 Agent 进行同步交互；显示最终答复和可读错误。
- `oryxos agent list|create|validate|archive`: 管理 Agent 目录与配置。
- `oryxos provider list|set|remove`: 管理供应商引用和启用状态。
- `oryxos health`: 输出服务健康状态。

CLI 与 REST 共用核心服务、权限策略和审计语义。
