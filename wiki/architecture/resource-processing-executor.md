# 资源处理执行器

这个页面记录源文档后半段拆解的执行器方法。它本质上是“通过代理拉取资源，然后根据能力与 MIME 类型把响应体送入责任链”的入口。

## 方法职责

输入：

- `Metadata metadata`
- `Request request`
- `ProxyPermit proxy`

输出：

- `ResourceState`

## 执行阶段

1. 基于现有 `OkHttpClient` 构造带代理认证的请求。
2. 同步执行 HTTP 请求。
3. 用响应码映射 `ResourceState`。
4. 如果 HTTP 访问失败，记录失败原因并快速返回。
5. 读取 `ResponseBody`。
6. 判空并校验 `contentLength` 是否超过上限。
7. 根据 `abilityId + originMediaType` 计算 `inboundMediaType`。
8. 解析 `HandlerStrategy`。
9. 如果没有匹配策略，返回 `ILLEGAL`。
10. 构造 `DefaultExecutionContext`。
11. 调 `strategy.chain.doHandle(context, strategy.inboundType)` 执行真正解析。

## 设计模式

- 策略模式：通过 `resolveHandlerStrategy(...)` 找到处理策略。
- 责任链模式：通过 `strategy.chain.doHandle(...)` 把工作交给处理链。

## 设计价值

- HTTP 访问层和资源处理层解耦。
- 出现响应码失败、body 为空、文件过大、类型不支持时快速失败。
- `metadata` 贯穿整个下载与解析链路，作为共享上下文。

## 潜在关注点

- `URI` 或能力映射规则变化后，策略匹配容易失准。
- `contentLength == -1` 时不会触发超大文件保护，需要依赖后续读取期控制。
- 这类入口非常依赖 `HandlerStrategy` 和责任链配置的完整性。

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
