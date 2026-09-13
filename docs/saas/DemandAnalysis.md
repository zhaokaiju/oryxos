# OryxOS SaaS 多租户需求文档

## 1. 项目目标

在保留 OryxOS Agent Runtime、Tool、Skill、Memory、Knowledge、审计和管理能力的基础上，支持多个企业租户安全共享平台资源。租户必须拥有独立的身份空间、数据空间、配置空间、资源配额和审计视图。

## 2. 角色

- **平台管理员**：管理平台节点、区域、套餐、全局策略和租户生命周期；默认不能读取租户业务内容。
- **租户管理员**：管理本租户成员、Agent、Provider、Tool、Skill、Knowledge、配额和审计权限。
- **租户成员**：在被授权范围内使用 Agent 和查看自己的会话。
- **租户审计员**：只读查询本租户审计与合规记录。
- **平台审计员**：查看平台级安全事件和租户元数据，不默认查看业务正文。

## 3. 核心需求

### 3.1 租户生命周期

系统 MUST 支持租户创建、启用、暂停、归档和删除。租户删除必须经过确认，执行数据导出、异步清理和删除证明；暂停租户不得发起模型、Tool 或定时任务调用。

### 3.2 身份与权限

系统 MUST 在每次请求中解析不可伪造的 `tenantId` 和主体身份。所有管理和运行操作 MUST 经过租户级 RBAC 校验；跨租户访问默认拒绝。平台管理员的运维权限与租户业务读取权限分离。

### 3.3 数据隔离

以下对象 MUST 绑定租户：Agent、Provider、Session、Memory、Knowledge、Skill 绑定、Tool Policy、Schedule、Audit Record、用量和账单。数据库、缓存、文件路径、索引、日志、指标和异步任务均 MUST 带租户上下文。

### 3.4 Provider 与密钥

租户可独立配置 Provider 和凭证引用。凭证 MUST 使用租户密钥空间或外部密钥系统保存；接口、日志、审计和平台运维页面不得返回明文。

### 3.5 Agent 执行

ReActLoop、ToolExecutor、SandboxChecker 和 ContextLoader MUST 在租户上下文中运行。Agent 只能发现本租户授权的 Skill、Tool、Memory 和 Knowledge；模型调用不得跨租户复用上下文。

### 3.6 配额与计量

系统 MUST 支持按租户限制并发会话、请求频率、Token、存储、Agent 数量和 Tool 执行次数。每次模型和 Tool 调用 MUST 产生租户维度的用量记录，可用于成本分析和计费。

### 3.7 审计与合规

系统 MUST 记录租户、主体、Agent、Session、Provider、Tool、策略结果、时间、来源和 traceId。租户只能查询自身记录；审计记录支持保留期限、导出和删除证明策略。

### 3.8 管理入口

REST、Web、CLI、定时任务和外部 Channel MUST 使用统一租户解析和权限语义。登录、租户切换、成员邀请、角色授权、配额查看、审计查询和数据导出属于 SaaS 管理流程。

## 4. 非目标

首期不实现任意租户自定义代码执行、跨租户 Agent 协作、自动跨区域迁移和复杂工作流编排。专属实例可作为后续部署模式，但必须复用同一租户数据和权限契约。

## 5. 验收指标

- 100% 的业务对象和执行请求可关联租户标识。
- 100% 的跨租户访问在业务处理前被拒绝。
- 100% 的模型、Tool、管理和安全事件具备租户维度审计记录。
- 新租户可在 10 分钟内完成成员、Provider 和首个 Agent 配置。
- 租户暂停后，100% 的新执行请求在 1 秒内被拒绝。
- 租户管理员无法读取其他租户的凭证、会话、Memory、Knowledge 或审计正文。

## 6. 关键实体

Tenant、TenantMembership、Role/Permission、TenantQuota、TenantKeyRef、TenantResource、UsageRecord、BillingAccount、AuditRecord、DataExportJob、DeletionJob。
