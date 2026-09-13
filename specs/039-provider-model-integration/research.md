# Research: Provider 模型统一接入

## Decision: 显式名称注册表

**Rationale**: 不同供应商通常实现同一模型接口，名称注册表能保证确定性路由、运行时替换和可测试性。

**Alternatives considered**: 按类型注入会产生 Bean 歧义；硬编码分支扩展成本高。

## Decision: 适配器隔离协议差异

**Rationale**: ProviderService 面向统一请求/响应，适配器负责协议字段、错误和用量转换，避免 Agent 依赖供应商细节。

**Alternatives considered**: 在调用方散落供应商判断会造成重复和不一致。

## Decision: 凭证引用而非凭证值

**Rationale**: 配置可持久化和可审计，同时将密钥生命周期交给环境或企业密钥系统。

**Alternatives considered**: 明文存储不可接受；启动时强制要求所有供应商凭证会降低管理可用性。

## Decision: 配置变更后刷新注册表

**Rationale**: 运维切换供应商无需重启或修改 Agent 指令，下一次请求即可使用最新配置。

**Alternatives considered**: 只在启动加载会造成配置陈旧和运维窗口。
