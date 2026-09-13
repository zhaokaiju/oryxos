# Quickstart Validation

## Prerequisites

- Java 21、Maven 3.9+
- 两个可用模型供应商的环境变量凭证

## Build

```bash
mvn verify
```

## Configure and route

创建 `provider-a` 与 `provider-b`，分别配置不同模型。通过 REST 或 CLI 发送带 `providerName` 的请求，验证响应包含统一字段和唯一 `callId`，并确认两次请求分别到达指定供应商。

## Maintenance and failures

停用 `provider-a` 后再次请求，预期请求不离开系统并返回明确错误。删除不存在或被 Agent 引用的供应商，预期操作被拒绝。清空凭证引用后测试连接，预期返回脱敏的凭证缺失错误。

## Acceptance evidence

记录路由命中、配置启停、切换后生效、错误脱敏和调用标识结果；检查日志和后续审计查询中不存在密钥值。
