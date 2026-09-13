# Provider REST Contract

统一错误格式：`{code, message, traceId, details}`；响应永不返回凭证明文。

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1/providers` | 列出供应商及启用状态 |
| POST | `/api/v1/providers` | 创建供应商配置 |
| PUT | `/api/v1/providers/{name}` | 更新连接、模型或凭证引用 |
| POST | `/api/v1/providers/{name}/enable` | 启用供应商 |
| POST | `/api/v1/providers/{name}/disable` | 停用供应商 |
| DELETE | `/api/v1/providers/{name}` | 删除未被活动引用的供应商 |
| POST | `/api/v1/providers/{name}/test` | 验证连接并返回脱敏结果 |

模型请求内部使用 `providerName`，返回 `callId`；不存在、停用、凭证缺失和连接失败使用不同可诊断错误码。
