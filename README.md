# HTC Agent User Layer

本仓库是 HTC Agent Q1 Web 用户层 Demo。目标是在 2026-07-10 前完成一轮 RAG 问答闭环验证：

```text
用户提问 -> Vue 前端 -> Java Spring Boot BFF -> Agent 层 -> 检索工具 -> 知识库 -> LLM -> 答案和引用展示
```

当前仓库已有 Vue + Vite 前端 Demo。Q1 最新架构要求 Web 层包含两部分：

- Vue 前端：聊天页面、输入发送、流式展示、引用展示、异常提示、复制和清空。
- Java Spring Boot BFF：Web API、Mock、Agent 转发、SSE 透传、异常映射、trace_id、日志和 Demo 兜底。

Web 层不直接实现检索、Prompt、LLM、Apache Tika 文档解析、分块、向量化或向量存储。

## 文档入口

开发前先读：

- `docs/agent.md`：给人和 AI agent 的开发指令。
- `docs/q1-three-phase-plan.md`：Q1 三阶段任务分工。
- `docs/api-contract.md`：Vue、Java Spring Boot BFF 和 Agent 的接口契约。
- `docs/vibe-guide.md`：每轮 vibe 开发时如何推进。
- `docs/july-demo-delivery-scope.md`：2026-07-10 Demo 必做、加分和暂缓功能。
- `docs/q1-web-architecture-refinement.md`：细化架构方案。

## Q1 三阶段

| 阶段 | 时间建议 | 目标 |
|---|---|---|
| 第一阶段 | 2026-06-18 到 2026-06-23 | 契约、骨架和 Mock 闭环 |
| 第二阶段 | 2026-06-24 到 2026-06-27 | 交互完善和异常补齐，任务量较少 |
| 第三阶段 | 2026-06-28 到 2026-07-10 | 真实联调、部署和 Demo 固化 |

## 前端快速启动

```bash
npm.cmd install
npm.cmd run dev
```

打开 Vite 输出的本地地址，默认通常是：

```text
http://127.0.0.1:5173/
```

## 前端测试

```bash
npm.cmd test
```

## Mock 场景

Q1 必须覆盖：

- 正常回答；
- 无相关文档；
- 检索异常；
- 模型异常；
- 网络异常；
- 请求超时；
- 流式中断；
- 引用无链接。

## BFF 接口

Vue 默认应调用 Java Spring Boot BFF：

```text
POST /api/v1/chat
```

请求体：

```json
{
  "query": "项目 Q1 阶段需要完成哪些功能？",
  "session_id": "local-session-001",
  "stream": true
}
```

响应体：

```json
{
  "trace_id": "trace-001",
  "status": "success",
  "answer": "Q1 阶段需要打通用户提问、检索、生成答案和展示引用的基础链路。",
  "citations": [
    {
      "index": 1,
      "title": "Q1 范围说明",
      "doc_id": "doc-q1-scope",
      "chunk_id": "chunk-001",
      "source_url": "/documents/doc-q1-scope"
    }
  ]
}
```

完整接口以 `docs/api-contract.md` 为准。

## 环境配置

当前前端支持 Mock 和真实接口切换。建议后续把真实地址指向 Spring Boot BFF，而不是直接指向 Agent。

```bash
VITE_AGENT_MODE=real
VITE_AGENT_API_URL=http://127.0.0.1:8080/api/v1/chat
```

如果真实 Agent 或 BFF 不稳定，保留 Mock 兜底能力，确保 Demo 可演示。

## Java Spring Boot BFF 约定

新增 BFF 时默认放在：

```text
server/
```

必须提供：

- `POST /api/v1/chat`
- 健康检查接口；
- Mock Profile 或配置开关；
- 统一 DTO；
- 统一异常映射；
- `trace_id` 日志；
- JSON 完整答案支持；
- SSE 透传或适配。

## Demo 最低成功标准

2026-07-10 前必须做到：

- 页面正常加载；
- 用户可以输入并发送问题；
- 空输入不会发送；
- 请求中不会重复提交；
- 系统能展示答案；
- 系统能展示 `trace_id`；
- 系统能展示 citations；
- `source_url` 为空时不生成无效链接；
- 能处理无相关文档、检索异常、模型异常、网络异常、超时和流式中断；
- 支持复制回答；
- 支持清空对话；
- 真实链路不可用时 Mock 仍能完成演示。
