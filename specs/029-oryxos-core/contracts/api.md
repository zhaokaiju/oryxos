# REST Contract

所有接口返回结构化 JSON，并使用统一错误格式 `{code, message, traceId, details}`。凭证字段只接受引用或脱敏值。

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/health` | 健康检查与版本状态 |
| GET/POST | `/api/v1/agents` | 查询或创建 Agent |
| GET/PUT/DELETE | `/api/v1/agents/{name}` | 查看、更新、归档 Agent |
| POST | `/api/v1/agents/{name}/chat` | 发起同步对话并返回最终结果 |
| GET | `/api/v1/sessions/{id}` | 查询会话与消息摘要 |
| GET/POST/PUT/DELETE | `/api/v1/providers` | 管理模型供应商配置 |
| GET | `/api/v1/audit` | 按 trace、session、agent 查询审计记录 |
| GET/POST/PUT/DELETE | `/api/v1/schedules` | 管理定时任务 |

工具拒绝、供应商失败和参数错误必须返回可诊断错误码，并保留对应审计记录。
