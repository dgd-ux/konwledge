# 回调与状态回收

回调阶段是真正统一 v1/v2 分流的地方，system 会根据 `ResultParam.distributeVersion` 决定采用哪套回调服务。

## 回调入口

runner 侧：

- `DefaultCompleteCallback.postProcess()`
- 组装 `ResultParam`
- 调 system 侧 `POST /callback`

system 侧：

- `CallbackController.receive()`
- `ServerBizImpl.callback()`

## 分流规则

- `distributeVersion == 2` 时进入 `CallbackServiceV2.receive()`
- 否则进入 `CallbackServiceV1.receive()`

## v1 回调特点

- 非火山海外云先释放代理 permit。
- 失败且可重试时重新入队，最多重试 3 次。
- 最终回调 ATSP、保存结果、移除 runner_task 关联、更新任务状态。

## v2 回调特点

- 先移除 `runner_task` 映射。
- 回收 runner/proxy 当前 permit。
- 释放代理 permit。
- 对 `V2_DISTRIBUTE_MEDIAS` 还会释放负载堆中的占用。
- 可重试失败时只把任务改回 `WAIT` 并增加 `retryTimes`。
- 最终成功或不可重试失败时才保存结果并回调 ATSP。

## 核心区别

- v1 重点是任务重入队和基础 permit 回收。
- v2 额外多了一层负载均衡状态回收，否则堆状态会失真。

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
