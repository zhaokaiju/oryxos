# Quickstart Validation

## Prerequisites

- Java 21、Maven 3.9+
- 一个可用模型供应商凭证，以环境变量提供

## Build and initialize

```bash
mvn verify
java -jar oryxos-boot/target/oryxos-boot-*.jar init
```

预期：工作区目录创建成功，重复执行不破坏已有 Agent。

## Provider and ReAct scenario

配置一个供应商后运行：

```bash
java -jar oryxos-boot/target/oryxos-boot-*.jar chat --profile default
```

输入需要外部信息的问题，预期 Agent 调用工具、回填结果并返回最终答复；达到迭代上限时安全结束。

## Memory and Skill scenario

在会话中保存一个偏好，开启新会话后询问同一偏好。为 Agent 创建指向公共 Skill 的相对软链接，预期提示词只显示名称、描述和路径；越界或悬空链接被拒绝。

## Sandbox and audit scenario

分别请求白名单内外的文件、命令和域名。预期越权请求在执行前拒绝；每次模型调用、工具调用和拒绝结果均可按 `traceId` 在审计查询中找到。

## REST scenario

启动服务后验证：

```bash
curl http://localhost:8080/api/v1/health
curl -X POST http://localhost:8080/api/v1/agents/default/chat \\
  -H 'Content-Type: application/json' -d '{"message":"hello"}'
```

预期：健康接口返回成功；对话接口返回最终结果、会话标识和可追踪信息。
