# v1 与 v2 调度分流

任务是否走 v1 或 v2，不是单看 `distributeVersion`，而是由任务来源、媒体类型、能力类型和具体调度入口共同决定。

## 走 v1 的典型条件

- 非直播场景。
- 不在 `Constants.V2_DISTRIBUTE_MEDIAS` 中的媒体类型。
- 仍依赖“按云厂商 + 资源类型分队列”调度模型的任务。

## v2 固定媒体集合

- `dy-picture`
- `dy-video`
- `xhs-picture`
- `xhs-video`

## 走 v2 的两类场景

### 1. 固定媒体类型

- 由 `TaskPuller` 拉取。
- 写入任务表。
- `DownloadTaskExecutor` 定时拉任务。
- 对每个任务调用 `DistributorV2.distribute(task)`。

### 2. 直播能力

- 由 `DistributeSchedule.labelDispatch()` 进入。
- 最终调用 `LiveDistributor.distribute()`。
- 直播能力 ID 在 `LiveDistributor.LIVE_ABILITY` 中硬编码。

## 为什么容易误解

- 任务落库时常常被写成 `distributeVersion = 2`。
- 但执行链路仍要看：
  - 是由 `TaskPullerV1` 还是 `TaskPuller` 拉入系统。
  - `ability.mediaId` 是否在 v2 固定媒体集合里。
  - `abilityId` 是否属于直播能力。

## 一句话判断

`distributeVersion` 更像回调分流标记，真正执行分流由拉取器和调度器拓扑决定。

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
