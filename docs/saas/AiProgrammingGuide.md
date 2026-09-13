# OryxOS SaaS 多租户 AI 编程指南

## 1. 实施方法

多租户是横切能力，采用 Spec-Kit 管理需求、方案、任务和一致性分析；每个阶段完成后运行安全测试和租户隔离验收。不得把 tenant_id 作为临时参数散落在业务代码中，必须先建立统一上下文和授权门面。

## 2. 推荐 user story 顺序

| User Story | 目标 | 依赖 |
|---|---|---|
| US-1 | Tenant、身份上下文与 RBAC | 无 |
| US-2 | 共享数据的强制租户隔离 | US-1 |
| US-3 | Agent Runtime、Tool、Memory、Skill 的租户化 | US-2 |
| US-4 | Provider 密钥、配额、计量与审计 | US-2、US-3 |
| US-5 | SaaS Web 控制面、导出删除和运营 | US-1~US-4 |

## 3. Spec-Kit 工作流

1. `/speckit-constitution`：确认租户隔离、默认拒绝、审计和凭证边界为不可协商原则。
2. `/speckit-specify`：每次只描述一个 user story，明确角色、租户边界和验收场景。
3. `/speckit-clarify`：优先澄清身份来源、共享/专属存储、删除和数据驻留等高风险决策。
4. `/speckit-plan`：输出 TenantContext、授权层、数据迁移、隔离策略和恢复方案。
5. `/speckit-tasks`：任务必须包含越权测试、串租户测试、审计和回滚任务。
6. `/speckit-analyze`：检查每条租户需求是否有任务覆盖，以及是否违反 OryxOS 宪法。

## 4. 编程门禁

- 所有入口 MUST 生成并验证 TenantContext。
- Repository、缓存、文件、索引和异步任务 MUST 自动附加租户边界。
- 禁止从请求体或可修改参数信任 tenant_id。
- 任何模型、Tool、Memory、Knowledge 和审计查询 MUST 经过统一授权服务。
- 测试必须包含“租户 A 不能读取或修改租户 B”的负向场景。
- 日志和错误必须脱敏；凭证只允许使用引用。
- 删除、迁移和导出必须具备幂等、可重试和状态记录。
- 不得启用 Spring AI 自动工具执行，不得绕过 SandboxChecker。

## 5. 每个 user story 的验收

### US-1

创建两个租户和不同角色，验证登录、租户切换、邀请、角色授权和暂停状态；伪造 tenant_id 必须被拒绝。

### US-2

对 Agent、Provider、Session、Memory、Knowledge、文件和审计执行跨租户读写尝试；验证数据库、缓存、文件路径和索引均隔离。

### US-3

分别在两个租户创建同名 Agent/Skill，验证 Prompt、ToolRegistry、Memory 和 Skill 正文不会串租户；检查软链接真实路径。

### US-4

验证租户密钥引用、配额耗尽、用量计量、模型/Tool 审计和平台管理员权限边界。

### US-5

验证租户导出、删除、恢复、区域策略和 Web 控制面；删除完成后不得继续接受该租户请求。

## 6. 风险与复盘

优先防止“默认租户”“后台任务无租户”“缓存键缺 tenant_id”“日志泄露业务正文”“平台管理员绕过授权”五类事故。每次实现完成后保留 Spec、Plan、Tasks、分析报告和隔离测试证据，供后续审计和社区协作复用。
