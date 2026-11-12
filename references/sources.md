# 实践来源与采用边界

本技能从 `app-workflow` 的五入口、单份 change、当前 PRD、按授权同步与历史归档改造。将平台交付改为能力/厂商/环境交付，移除移动端固定目录和 UI 设计流程；增加运行 Harness、LLM 协议适配、效果评测及后端状态边界。它是工作方法，不是某个框架的教程。

以下公开来源于 **2026-09-21** 查阅。本文概括采用原则；具体模型能力、价格、字段和 SDK 行为必须在实际接入时重查官方版本，不能从本文推断。

## Anthropic：从简单可组合的 Agent 开始

[Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)

采用：区分确定性 workflow 与动态 Agent，优先简单流程；工具接口要清晰，运行有明确停止条件。落在 [架构原则](architecture-principles.md) 和 [Harness](harness.md)。不强制采用该厂商模型、SDK 或多 Agent，文章里的框架清单不作为选型结论。

## Anthropic：长任务的 Harness 与上下文

[Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)

采用：跨上下文的任务不能只靠模型记忆；需要外部状态、可续接进度和从真实入口验证结果。落在运行状态、恢复和 build 的切片验证。原文聚焦长时编码 Agent，不把其 initializer/coding agent 双角色、进度文件或特定清单格式套成聊天后端强制机制。

[Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

采用：上下文是有限资源，按任务选择工具和资料，有界输出与压缩需保留关键信息。落在 [Harness 的上下文部分](harness.md)。检索、长期记忆和多 Agent 仅作为有真实需求时的选择。

## Anthropic：评测整个 Agent 的结果

[Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)

采用：区分任务、trial、轨迹和环境最终结果；组合代码/模型/人工评分，观察非确定性和失败切片，从实际人工检查与失败案例开始。落在 [测试与评测](evaluation.md) 和 EVAL 模板。不把示例用例数量、指标或某次成功写成所有项目的固定门槛。

## LangGraph：持久化与故障恢复

[Persistence](https://docs.langchain.com/oss/python/langgraph/persistence) · [Fault tolerance](https://docs.langchain.com/oss/python/langgraph/fault-tolerance)

采用：会话内检查点与跨会话记忆不同；重试、超时、失败处理和恢复有明确边界。查阅时旧 durable-execution 路径已重定向至 persistence，故使用现行链接。落在 [Harness](harness.md) 的状态/恢复及 [providers](providers.md) 的有界重试。

本技能对外部副作用幂等、结果未知窗口和并发所有权的要求是面向自研后端的工程综合，不表示 LangGraph 自动提供所有业务保证，也不要求安装 LangGraph。

## OpenTelemetry：关联模型调用与执行信号

[OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai)

采用：用 spans、metrics、events 表达模型/工具相关执行并保留 provider 信息。查阅时原站 GenAI 文档已迁往此独立仓库。落在 Harness 可观测性和评测证据；字段与稳定性按项目使用的规范版本核对，不把草案字段写死，不要求记录敏感正文或安装特定观测平台。

## OWASP：模型不能成为授权边界

[LLM01: Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)

采用：把外部内容视为不可信数据，最小权限由服务端执行，确需高风险动作的产品可以设置绑定具体操作的确认。落在工具权限与相关回归用例。提示词、RAG 或分隔标签不能单独保证防注入，也不因此对普通开发动作增加无关审批。

这些来源用于说明决策依据，不替代目标项目当前约束。使用技能时按实际任务读取相关资料，不要求每次重新阅读全部文章。
