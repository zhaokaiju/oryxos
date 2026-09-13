# Implementation Plan: OryxOS Agent OS 核心运行时

**Branch**: `029-oryxos-core` | **Date**: 2026-09-13 | **Spec**: [spec.md](spec.md)

## Summary

交付 OryxOS 单节点 Agent OS 核心运行时：统一模型供应商、可控 ReAct 循环、会话与记忆、工具与沙箱、审计，以及 CLI/Web 管理入口。实现按五个用户故事推进，先完成 Provider 与 ReAct 基础，再并行完成 Memory/Skill 与 Tool/Sandbox，最后提供管理接口和端到端验收。

## Technical Context

**Language/Version**: Java 21

**Primary Dependencies**: Spring Boot 3.x、Spring AI Alibaba、Maven 多模块、Picocli、SQLite、Flyway、Vue 3 管理台

**Storage**: SQLite 默认；通过迁移脚本管理结构；Agent 配置与 Skill 资源存于 `.oryxos/` 文件树

**Testing**: JUnit 5、Spring Boot Test、端到端 CLI/HTTP 验收；`mvn verify` 运行格式、静态分析和依赖安全检查

**Target Platform**: 企业自有 Linux 服务器、虚拟机、Kubernetes 或物理机

**Project Type**: 单可执行文件的 Spring Boot 多模块 Web 服务与 CLI

**Performance Goals**: 简单请求 P95 在 5 秒内返回（不含外部模型网络波动）；支持并发会话和虚拟线程

**Constraints**: 数据默认留在企业边界；核心循环同步执行；工具必须经过沙箱；凭证不得明文暴露；核心阶段不引入异步编程模型或复杂工作流编排

**Scale/Scope**: 单企业单实例、多个 Agent 和供应商；多租户、跨节点调度、流式输出和知识图谱留作扩展

## Constitution Check

*GATE: Must pass before Phase 0 research and after Phase 1 design.*

- [x] 自实现 ReAct 循环，项目控制工具调度与终止条件。
- [x] Spring AI 仅用于协议转换和 Schema 生成，禁用自动 tool 执行。
- [x] Provider 使用显式名称映射，不依赖同类型 Bean 扫描路由。
- [x] Agent 目录与本地相对 Skill 软链接是唯一绑定真相源，正文按需读取。
- [x] 模型调用和工具调用从第一天写入持久化审计记录。
- [x] 文件、命令、网络和软链接访问执行真实路径安全校验。
- [x] 核心同步执行并使用 Java 21 虚拟线程，不引入 Reactor/WebFlux/CompletableFuture。
- [x] Agent 实例无状态，状态外置；迁移由 Flyway 管理。
- [x] 复杂度无违反项需要豁免。

## Project Structure

### Documentation (this feature)

```text
specs/029-oryxos-core/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── checklists/requirements.md
```

### Source Code (repository root)

```text
oryxos-core/       # ReActLoop、Session、Profile、ContextLoader、OryxTool 抽象
oryxos-provider/   # ProviderService 与显式 ChatModel 映射
oryxos-memory/     # MemoryService、长期记忆与记忆工具
oryxos-tool/       # 内置工具、MCP Client、ToolRegistry、SandboxChecker
oryxos-channel-cli/# CLI Channel
oryxos-web/        # REST API 与 Web 管理台
oryxos-storage/    # SQLite 仓储、Flyway、审计记录
oryxos-cli/        # Picocli 命令与配置加载
oryxos-boot/       # Spring Boot 启动与聚合
```

**Structure Decision**: 采用技术方案定义的 9 个 Maven 模块；跨模块契约放在 `oryxos-core`，实现模块依赖接口，禁止循环依赖。核心工具能力合并在 `oryxos-tool`。

## Complexity Tracking

无宪法例外。
