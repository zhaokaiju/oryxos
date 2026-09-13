# Data Model

## Provider

- `name`: 实例内唯一标识；非空、长度受限、禁止路径分隔符。
- `protocol`: 协议/适配器类型。
- `baseUrl`: 服务地址，须符合允许的 URL 格式。
- `model`: 默认模型标识。
- `credentialRef`: 环境变量名或企业密钥引用，禁止保存明文值。
- `enabled`: 是否参与路由。
- `createdAt`, `updatedAt`: 配置时间。

状态：draft → enabled ↔ disabled → removed。删除前必须确认不存在活动引用。

## ModelRequest

- `providerName`, `messages`, `parameters`, `callId`, `createdAt`。
- `callId` 全局唯一并贯穿错误、日志和审计。

## ModelResponse

- `callId`, `content`, `toolCalls`（仅描述，不自动执行）、`usage`, `status`, `error`。

## Relationships

Provider 1—N ModelRequest；ModelRequest 1—1 ModelResponse。Agent profile 通过 `providerName` 引用 Provider。
