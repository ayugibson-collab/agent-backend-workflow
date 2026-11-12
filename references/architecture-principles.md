# Agent 后端架构原则

这些是默认建议，不能覆盖项目已接受的架构。目标是在自研 Harness 中清楚掌握执行、状态与协议边界，而不是重新实现成熟 HTTP 客户端、数据库驱动或协议 SDK。

## 从最简单的可用形态开始

普通聊天可能只需一次模型调用；固定业务流程优先确定性编排；只有动态工具选择有价值时才增加 Agent 循环。默认一个后端服务配所需存储即可；有明确隔离、吞吐或长任务需求再考虑 worker/队列。多 Agent、向量库、MCP 和长期记忆都是按需能力。

推荐逻辑关系：

```text
API / streaming transport
  -> application service: identity, session ownership, run lifecycle
     -> Harness: context -> model -> tool intent -> loop / finish
        -> provider port -> vendor adapter -> vendor API
        -> tool executor -> business service / external API
     -> state repository / event publication / telemetry
```

这是依赖说明，不是必须照搬的源码目录或微服务拆分。

## 边界与所有权

- API 层把请求/断线转换为明确协议事件；不持有唯一的业务执行状态，不让 SSE 连接对象决定全部 run 生命周期。
- 应用服务确定认证主体、会话归属、请求去重与运行策略。tenant/user 不取模型参数作为可信身份；单租户项目沿用实际身份边界，不硬建多租户平台。
- Harness 控制执行循环、上下文、预算与停止条件，依赖标准化模型/工具接口，不依赖特定 Web 框架请求对象。
- Provider adapter 翻译厂商消息、流事件、能力和错误，不决定业务权限、不直接执行工具。
- 工具执行器校验参数、授权、审批状态和幂等，再调用业务能力。读写业务规则留在业务服务，避免在工具里复制一套。
- 持久化存会话事实、必要的运行进度和执行结果；日志不充当恢复所需的唯一存储。HTTP 连接、运行取消和工具结果需由明确规则关联。
- 装配层选择具体 provider、存储和工具。只在实际 I/O 或替换边界抽象，不为每个实体生成 Repository/UseCase/接口。

## 并发与运维按需设计

同一会话并发写入会导致历史交错，需按产品约定拒绝、排队或分支，并在数据库/运行拥有权处保证；只在进程内加锁不能保护多副本。选择实际需要的并发控制，不默认分布式锁。

对长任务，明确执行 owner、租约失效、恢复后旧 worker 是否还能提交结果；需要时使用版本条件更新或 fencing。队列通常会重投递，业务幂等不能省略。没有这些机制时明确“不保证崩溃续跑”。

接入第三方模型时复用连接池和项目限流。按实际瓶颈限制全局/用户/provider 并发与排队，流式连接缓冲需要有界；不能用无限创建任务掩盖吞吐不足。

可观测性优先能定位问题的字段：请求/run/attempt/tool 的关联、版本、耗时、错误类型、结束原因和 token/成本来源。沿用项目日志/tracing，不因原则建议就安装一整套观测平台。详细运行不变量见 [Harness](harness.md)。
