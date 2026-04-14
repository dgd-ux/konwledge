# 下载调度链路笔记

这份笔记对应 `niop-open-download` 的任务调度说明，重点是 v1/v2 并存、任务如何进入系统、runner 如何执行、结果如何回写。

## 核心结论

- 项目不是单链路升级，而是 v1、v2 同时存在。
- 新任务提交时通常会落成 `distributeVersion = 2`，但真正走哪套执行链路不能只看这个字段。
- 真正的执行分流发生在拉取阶段和回调阶段。

## 主体结构

- 任务入口：外部提交、ATSP 被动拉取。
- 调度分流：`TaskPullerV1/DistributorV1` 对应传统链路，`TaskPuller/DownloadTaskExecutor/DistributorV2` 对应新链路。
- 直播场景：单独由 `DistributeSchedule -> LiveDistributor` 处理。
- runner 执行：system 下发任务到 runner，runner 下载资源并回调。
- 回调处理：system 依据 `ResultParam.distributeVersion` 进入 `CallbackServiceV1` 或 `CallbackServiceV2`。

## 扩展补充

源文档后半部分还额外拆解了一个“带代理发起 HTTP 请求并把响应交给处理链”的执行器方法，适合单独作为资源处理入口页保存。

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
