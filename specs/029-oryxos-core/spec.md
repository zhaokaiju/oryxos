# Feature Specification: OryxOS Agent OS 核心运行时

**Feature Branch**: `029-oryxos-core`

**Created**: 2026-09-13

**Status**: Draft

**Input**: User description: 基于 IndustryResearch、DemandAnalysis、TechnicalSolution、AiProgrammingGuide，定义 OryxOS 核心 Agent OS 能力。

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 统一接入大语言模型 (Priority: P1)

作为企业开发者，我希望用统一配置接入多个模型供应商，让 Agent 无需感知供应商差异即可完成对话。

**Why this priority**: 模型调用是所有 Agent 能力的基础，也是私有部署和供应商可替换的前提。

**Independent Test**: 配置至少两个供应商并分别发起请求，验证按名称选择模型、返回统一消息，并在凭证缺失时得到清晰错误。

**Acceptance Scenarios**:

1. **Given** 已配置有效供应商，**When** Agent 发起对话，**Then** 系统返回统一格式的模型响应。
2. **Given** 配置多个供应商，**When** 指定供应商名称，**Then** 请求只路由到该供应商。
3. **Given** 凭证缺失或供应商不可用，**When** 发起请求，**Then** 系统报告可诊断错误且不泄露凭证。

### User Story 2 - 可控的 ReAct 工具循环 (Priority: P1)

作为 Agent 使用者，我希望 Agent 能自主决定何时调用工具、读取结果并继续推理，直到给出最终答复。

**Why this priority**: ReAct 循环是 Agent 从文本生成走向可靠执行的核心机制。

**Independent Test**: 让 Agent 完成一次需要天气查询的任务，验证“思考—调用—回填—再思考—答复”链路、终止条件和迭代上限。

**Acceptance Scenarios**:

1. **Given** 工具已注册，**When** 模型返回工具调用，**Then** 系统执行一次工具并将结果回填到后续推理。
2. **Given** 模型返回最终答复，**When** 循环处理该响应，**Then** 系统结束循环并返回答复。
3. **Given** 循环达到迭代上限或工具失败，**When** 继续执行，**Then** 系统安全终止并返回可解释状态。

### User Story 3 - 跨会话记忆与 Skill 能力复用 (Priority: P2)

作为业务人员，我希望 Agent 记住经过确认的偏好，并按需复用企业沉淀的 Skill 与知识。

**Why this priority**: 记忆和能力沉淀让 Agent 从一次性问答变成可持续的企业资产。

**Independent Test**: 在一个会话保存偏好，在新会话召回并据此响应；绑定 Skill 后仅注入元数据，按需读取正文。

**Acceptance Scenarios**:

1. **Given** 用户明确要求记住偏好，**When** 会话结束并开启新会话，**Then** Agent 能召回该偏好。
2. **Given** Agent 绑定公共 Skill，**When** 组装提示词，**Then** 只注入有效绑定的名称、描述和路径。
3. **Given** Skill 链接越界、悬空或目标无效，**When** 加载 Agent，**Then** 系统拒绝该绑定并给出诊断信息。

### User Story 4 - 安全工具与审计 (Priority: P2)

作为企业管理员，我希望所有文件、命令和网络访问受沙箱限制，并且模型调用和工具调用可追溯。

**Why this priority**: 私有部署进入生产环境必须满足最小权限、合规留证和事故追查要求。

**Independent Test**: 分别执行允许与拒绝的文件、命令、域名请求，检查审计记录是否完整且敏感信息被保护。

**Acceptance Scenarios**:

1. **Given** 请求访问白名单之外的资源，**When** 工具执行前校验，**Then** 操作被拒绝且不触达外部资源。
2. **Given** 工具或模型调用成功或失败，**When** 调用完成，**Then** 对应审计记录写入持久化存储。
3. **Given** 请求通过软链接访问文件，**When** 校验真实路径，**Then** 越界目标被拒绝。

### User Story 5 - Web、CLI 与多端点管理 (Priority: P3)

作为运维和业务人员，我希望通过 CLI 或 Web 管理 Agent、供应商、会话和定时任务，并通过统一入口使用 Agent。

**Why this priority**: 管理入口将底座能力交付给非开发人员，并支持企业日常运营。

**Independent Test**: 初始化工作区、创建 Agent、发起同步请求、查询会话和健康状态，验证 CLI 与 Web 返回一致的结果。

**Acceptance Scenarios**:

1. **Given** 空工作区，**When** 执行初始化命令，**Then** 系统创建所需目录并可立即创建 Agent。
2. **Given** 已配置 Agent，**When** 通过 Web 或 CLI 发起请求，**Then** 返回同一 Agent 的最终结果并保留会话上下文。
3. **Given** 管理员查询健康、会话或审计信息，**When** 请求对应入口，**Then** 返回权限范围内的结构化数据。

### Edge Cases

- 模型返回无效工具名称、格式错误参数或空响应时，系统必须拒绝该调用并保留诊断上下文。
- 工具超时、网络中断、供应商限流或重复请求时，系统必须在可配置范围内安全失败，不产生重复副作用。
- Agent 配置、Skill 链接或迁移文件损坏时，启动恢复必须报告具体对象并继续保护其他 Agent。
- 并发会话写入同一记忆或审计记录时，结果必须保持一致且不丢失调用记录。
- 未配置模型凭证时，管理入口可以启动，但发起模型请求必须返回明确配置提示。

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系统 MUST 支持以供应商名称选择模型，并向 Agent 提供统一消息格式。
- **FR-002**: 系统 MUST 由自身控制 ReAct 循环、工具调度、迭代上限和终止条件。
- **FR-003**: 系统 MUST 支持会话历史、长期记忆、Skill 元数据注入及按需读取 Skill 正文。
- **FR-004**: 系统 MUST 将 Agent 定义为可发现、可更新的目录配置，并支持生命周期管理。
- **FR-005**: 系统 MUST 对文件路径、命令首 token 和网络域名执行白名单沙箱校验。
- **FR-006**: 系统 MUST 对允许软链接的访问校验真实路径，拒绝绝对、链式越界和无效目标。
- **FR-007**: 系统 MUST 持久化每次模型调用和工具调用的审计记录，包括成功、失败和拒绝结果。
- **FR-008**: 系统 MUST 通过环境变量或企业密钥体系提供凭证，禁止明文出现在代码、配置、日志和审计内容中。
- **FR-009**: 系统 MUST 提供 CLI 与 Web 管理入口，覆盖工作区初始化、Agent、供应商、会话、健康和定时任务管理。
- **FR-010**: 系统 MUST 在同步执行模型下支持并发会话，并在错误时返回稳定、可诊断的结果。
- **FR-011**: 系统 MUST 支持通过标准协议接入外部工具与通知能力，并保持工具来源对运行循环透明。
- **FR-012**: 系统 MUST 提供结构化日志、健康检查和可观测指标，且数据默认留在企业部署边界内。

### Key Entities *(include if data involved)*

- **Agent**：由目录、运行配置、任务指令、Skill 绑定和附属资源组成的可运行单元。
- **Provider**：模型供应商连接配置及其可用模型路由信息。
- **Session**：一次或多次对话的消息历史、关联 Agent 和运行状态。
- **Memory**：跨会话保存的偏好、事实和可检索内容。
- **Tool**：具有名称、描述、输入约束和执行结果的可调用能力。
- **Audit Record**：模型调用、工具调用、策略判定和结果的不可省略记录。

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 新用户可在 10 分钟内完成工作区初始化、配置一个 Agent 并发起首个成功请求。
- **SC-002**: 在标准测试环境中，95% 的简单对话请求在 5 秒内返回最终结果（不含外部模型网络波动）。
- **SC-003**: 100% 的工具调用和模型调用都有可关联的持久化审计记录，包含成功与失败路径。
- **SC-004**: 100% 的越权文件、命令、域名和软链接访问在执行前被拒绝。
- **SC-005**: 至少 90% 的受测用户可独立完成 Agent 创建、会话发起和运行记录查询。
- **SC-006**: 更换模型供应商配置无需修改 Agent 任务指令，核心验收场景仍可通过。

## Assumptions

- 首个交付阶段面向单企业、单实例私有部署；多租户、SSO、跨节点调度和复杂编排属于后续扩展。
- 用户具备至少一个可用模型供应商凭证；无凭证时系统仍可启动并提供管理能力。
- 外部工具优先通过标准协议接入；业务方负责其工具服务的业务授权和可用性。
- 核心执行采用同步交互；流式输出、复杂工作流编排和知识图谱属于后续阶段。
- 企业负责提供部署环境、密钥管理和白名单初始策略。
