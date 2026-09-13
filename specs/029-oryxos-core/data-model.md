# Data Model

## Agent

- `name`: 唯一标识，映射 `.oryxos/agents/<name>/`，禁止路径穿越。
- `profile`: 模型供应商、模型名、运行参数和启用状态。
- `instructions`: `AGENT.md` 正文；每次组装提示词时读取。
- `skillBindings`: Agent `skills/` 下的受控相对软链接集合。
- 生命周期：discovered → active → archived/deleted；加载失败时保留诊断状态。

## Provider

- `name`: 实例内唯一名称。
- `type/baseUrl/model`: 连接与模型路由信息。
- `credentialRef`: 环境变量或企业密钥引用，不保存明文凭证。
- `enabled`: 是否可被路由。

## Session

- `id`: 全局唯一标识。
- `agentName`: 关联 Agent。
- `messages`: 有序的用户、助手、工具消息。
- `status`: active、completed、failed。
- `createdAt/updatedAt`: 生命周期时间。

## Memory

- `id`, `agentName`, `content`, `kind`, `createdAt`, `updatedAt`。
- `kind` 至少支持 preference、fact、note；内容来自用户明确保存或系统确认。
- 通过 Agent 与 Session 关联，可跨会话召回。

## Tool

- `name`: 全局注册名称。
- `description`: 模型可见描述。
- `inputSchema`: JSON Schema。
- `source`: builtin、mcp 或 native。
- `execute`: 返回成功标识、内容、错误和可重试标记。

## AuditRecord

- `id`, `traceId`, `sessionId`, `agentName`, `kind`（llm_call/tool_invocation）。
- `requestSummary`, `resultSummary`, `status`, `durationMs`, `createdAt`。
- 敏感字段脱敏；成功、失败、拒绝均必须落库。

## Relationships

- Agent 1—N Session；Agent 1—N Memory；Session 1—N AuditRecord。
- Provider 1—N Agent profile references；Tool N—N Agent visibility via registry and policy。
