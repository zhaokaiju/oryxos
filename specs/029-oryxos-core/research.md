# Research: OryxOS Agent OS 核心运行时

## Decision: 采用显式 Provider 注册表

**Rationale**: 多供应商实现通常共享同一模型接口，按名称维护映射可避免容器扫描歧义，并支持运行时更新配置。

**Alternatives considered**: 按类型自动注入会在多供应商时产生不确定路由；按供应商写死分支则难以扩展。

## Decision: 项目自建同步 ReAct 循环

**Rationale**: 循环、迭代上限、上下文裁剪和工具执行必须可观测、可测试并由底座统一控制。

**Alternatives considered**: 使用框架自动 tool calling 会削弱控制权并可能造成重复执行；异步模型增加核心链路复杂度。

## Decision: Agent 文件树 + Skill 相对软链接渐进披露

**Rationale**: 目录即 Agent，业务方无需编写后端代码；每轮只注入 Skill 元数据，正文按需读取可控制上下文成本。

**Alternatives considered**: 复制 Skill 内容会产生漂移；在 frontmatter 维护第二份索引会与目录绑定冲突。

## Decision: SQLite 与 Flyway 管理核心状态和审计

**Rationale**: 单节点私有部署易安装；迁移脚本可审查、可重复执行，满足审计记录不可省略的要求。

**Alternatives considered**: Hibernate 自动建表对演进和 SQLite ALTER TABLE 支持不足；仅写日志无法可靠追溯。

## Decision: 四类沙箱校验与真实路径判定

**Rationale**: 文件、命令、域名和软链接分别面对不同越权面；真实路径校验能阻断软链接绕过字符串白名单。

**Alternatives considered**: `normalize()+startsWith()` 无法处理已存在软链接；JDK SecurityManager 已不可用。

## Decision: CLI 与 REST/Web 共用同一核心服务

**Rationale**: 统一服务层可保证入口行为一致，CLI 适合初始化和运维，Web 适合日常管理。

**Alternatives considered**: 入口各自实现会导致权限、错误和审计语义分叉。
