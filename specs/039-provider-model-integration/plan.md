# Implementation Plan: Provider 模型统一接入

**Branch**: `039-provider-model-integration` | **Date**: 2026-09-13 | **Spec**: [spec.md](spec.md)

## Summary

建立显式的供应商注册与路由服务，将不同模型协议转换为统一请求/响应；支持供应商 CRUD、启停、凭证引用、运行时刷新与调用标识，并为后续审计和 Agent 循环提供稳定接口。

## Technical Context

**Language/Version**: Java 21

**Primary Dependencies**: Spring Boot 3.x、Spring AI Alibaba、Spring Data JPA、Flyway、Maven

**Storage**: SQLite 中持久化供应商配置；凭证只保存环境变量或企业密钥引用

**Testing**: JUnit 5、Spring Boot Test、模拟供应商路由测试、REST/CLI 集成测试

**Target Platform**: 企业私有部署的 Linux/Kubernetes/虚拟机

**Project Type**: Spring Boot 多模块服务，提供 REST 与 CLI 管理能力

**Performance Goals**: 正常请求 P95 ≤ 5 秒（不含外部网络波动）；配置更新后下一请求使用新配置

**Constraints**: 显式名称映射；禁止自动 Tool 执行；凭证脱敏；同步调用；模型调用须带 trace/call ID

**Scale/Scope**: 单实例多个供应商；本特性不包含 ReAct、流式输出和自动故障转移

## Constitution Check

- [x] Provider 使用显式 `name → ChatModel` 映射。
- [x] Spring AI 仅用于协议转换和 Schema 生成，不启用自动工具执行。
- [x] 凭证通过环境变量/企业密钥引用读取，不明文持久化或输出。
- [x] 模型调用生成可关联标识并接入后续审计。
- [x] 使用 Java 21 同步执行和虚拟线程边界。
- [x] 状态外置并通过迁移脚本管理。
- [x] 无需宪法例外。

## Project Structure

```text
oryxos-provider/src/main/java/**/ProviderConfig.java
oryxos-provider/src/main/java/**/ProviderRegistry.java
oryxos-provider/src/main/java/**/ProviderService.java
oryxos-provider/src/main/java/**/ProviderAdapter.java
oryxos-provider/src/test/java/**/ProviderServiceTest.java
oryxos-storage/src/main/java/**/ProviderRepository.java
oryxos-web/src/main/java/**/ProviderController.java
oryxos-cli/src/main/java/**/ProviderCommand.java
```

**Structure Decision**: Provider 领域逻辑归 `oryxos-provider`；持久化归 `oryxos-storage`；Web/CLI 只调用共享服务，不重复路由逻辑。

## Complexity Tracking

无宪法例外。
