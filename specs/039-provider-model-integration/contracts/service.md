# Provider Service Contract

- `ModelResponse call(ModelRequest request)`: 按 `providerName` 显式路由；生成 `callId`；不执行返回的工具调用。
- `ProviderConfig create/update/enable/disable/remove(...)`: 校验名称、地址和凭证引用；变更后刷新注册表。
- `ProviderStatus test(String providerName)`: 使用安全探测请求验证连接，结果脱敏并可审计。
