# OryxOS SaaS 多租户技术方案

## 1. 设计原则

1. **租户上下文不可丢失**：从入口到 ReActLoop、Tool、存储和审计全链路传递 `TenantContext`。
2. **默认拒绝**：任何缺失、伪造或不一致的租户上下文都拒绝处理。
3. **控制面与数据面分离**：平台管理租户和资源生命周期，租户数据面执行 Agent 请求。
4. **共享运行时、隔离状态**：实例可共享，状态、密钥、文件和索引按租户隔离。
5. **宪法约束不变**：自实现 ReAct、显式 Provider 映射、同步执行、真实路径校验和 Day One 审计继续适用。

## 2. 架构

### 2.1 租户上下文

`TenantContext` 包含 `tenantId`、主体 ID、角色、区域、traceId 和请求来源。REST 从认证令牌解析，CLI 从登录会话解析，定时任务从任务归属解析，Channel 从绑定配置解析。禁止从请求体接受可信 `tenantId`。

### 2.2 分层

- **SaaS Gateway**：认证、租户解析、限流、请求路由和审计入口。
- **Tenant Control Plane**：租户、成员、角色、套餐、配额、区域和导出删除任务。
- **Agent Runtime**：复用 OryxOS Core、Provider、Memory、Tool、Knowledge 和 Channel 模块。
- **Tenant Storage**：关系数据采用共享表强制 `tenant_id`、按租户 schema 或专属数据库；文件路径采用租户根目录；向量索引采用租户命名空间。
- **Metering**：模型、Tool、存储和请求计量写入不可变用量表。

## 3. 数据隔离策略

首期采用共享数据库、所有租户表强制 `tenant_id`、Repository 自动注入租户谓词、数据库行级安全作为第二道防线。高合规租户支持专属 schema/数据库。所有唯一索引包含 `tenant_id`，例如 `(tenant_id, name)`。

文件根目录：`.oryxos/tenants/<tenantId>/...`。SandboxChecker 先校验租户根，再校验 Agent/Skill 的真实路径；禁止通过软链接跨租户或跨工作区。

缓存键、消息主题、定时任务、指标标签和日志字段必须包含租户标识；日志默认只记录租户元数据和脱敏摘要。

## 4. 身份、权限与密钥

采用 OIDC/OAuth2 兼容身份层；令牌中的租户和角色声明需经过签名验证。`TenantAuthorizationService` 统一执行资源归属和动作权限判断。平台管理员使用独立运维权限，不能绕过租户数据访问策略。

Provider 凭证保存为 `tenant_key_ref`，解析时使用租户密钥空间；应用内存中短暂存在，禁止写入数据库、日志、审计和错误响应。

## 5. Agent 执行改造

`ReActLoop` 创建请求级 `TenantContext`，`PromptBuilder` 只加载当前租户 Agent、Skill、Memory 和 Knowledge。`ToolRegistry` 按租户和策略过滤能力；`AuditService` 为每个 LLM/Tool 调用写入 tenant_id、主体、traceId 和策略结果。租户配额在执行前预留、执行后结算，失败时释放未使用额度。

## 6. 控制面接口

- `POST/GET /api/v1/tenants`：平台租户生命周期
- `GET/POST /api/v1/tenants/{id}/members`：成员和角色
- `GET/PUT /api/v1/tenant/quota`：租户配额
- `POST /api/v1/tenant/export`：租户数据导出
- `POST /api/v1/tenant/deletion`：删除申请与证明
- 现有 Agent、Provider、Session、Audit 接口均从认证上下文获取 tenant_id，不再信任客户端字段。

## 7. 可靠性与合规

租户暂停状态由缓存加数据库双重判定；执行前检查配额和状态。备份按租户可恢复；导出包加密并设置过期时间。删除任务采用可重试状态机，记录发现、导出、清理、校验和完成证明。跨区域部署遵循数据驻留策略。

## 8. 迁移与演进

为现有单租户数据建立默认租户并回填 `tenant_id`；迁移必须可回滚、可分批执行并通过一致性校验。所有新表和 Repository 从第一版即要求租户字段和过滤器，禁止后补隔离。

## 9. 监控与安全验证

监控租户级请求量、拒绝数、配额耗尽、跨租户访问告警、密钥访问和删除任务。自动化测试覆盖 Repository 越权、缓存键碰撞、文件软链接越界、异步任务串租户和平台管理员越权。
