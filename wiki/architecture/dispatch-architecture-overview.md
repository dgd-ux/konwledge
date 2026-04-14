# 调度架构总览

`niop-open-download` 的真实结构不是一条下载链路，而是 v1 和 v2 两套任务调度体系并存。

## 系统中同时存在的组件

- `TaskPullerV1`：v1 拉取链路
- `TaskPuller`：v2 拉取链路
- `DistributorV1`：v1 调度器
- `DownloadTaskExecutor`：普通 v2 执行器
- `DistributeSchedule -> LiveDistributor`：直播场景 v2 调度入口

## 任务入口

### 外部主动提交

- `TaskController`
- `TaskService.submit/submitBatch`
- `AtspTaskProcessor.submitTask/submitTasks`

新提交流程里，`AtspTaskProcessor.buildTask()` 会把 `DistributeTask.distributeVersion` 设为 `2`。

### ATSP 被动拉取

- `TaskPullerV1` 负责老链路拉取。
- `TaskPuller` 负责新链路拉取。

## 架构特征

- v1 偏向“按资源类型分桶后批量调度”。
- v2 偏向“按任务粒度逐个调度，并结合 groupId 做负载均衡”。
- 直播虽然也属于 v2 思路，但走独立的调度入口。

## 关键认识

- `distributeVersion` 不能单独决定执行器。
- 真正入口由 puller、distributor、executor 的组合决定。
- 回调阶段才会再次通过 `distributeVersion` 进入对应的回调服务。

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
