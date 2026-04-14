# 核心对象速查

这页只保留源文档中最值得盯的对象和字段，用于快速回忆 system、runner、callback 三段链路的关键数据。

## DistributeTask

- `taskId`：任务唯一标识
- `abilityId`：下载能力
- `appId`：应用配置入口
- `configId`：上传配置、`groupId` 等的定位键
- `mediaId`：资源类型，也是 v1/v2 覆盖范围的重要依据
- `groupId`：runner/proxy 资源池
- `csp`：云厂商
- `state`：`WAIT / PROCESS / SUCCESS / FAILED`
- `retryTimes`：是否重试
- `distributeVersion`：回调分流关键字段

## DownloadTask

- `id`
- `abilityId`
- `appId`
- `configId`
- `url`
- `headers`
- `duration`
- `path`
- `proxyPermit`
- `retryTimes`
- `distributeVersion`

## ResultParam

- `taskId`
- `abilityId`
- `resultView`
- `proxyPermit`
- `runnerIp`
- `retryTimes`
- `downloadRecord`
- `distributeVersion`

## RunnerParam

- `ip`
- `port`
- `csp`
- `taskAmount`
- `hostName`
- `runnerVersion`

## RunnerDTO

- `address`
- `ip`
- `maxPermits`
- `currentPermits`
- `state`
- `csp`

## ProxyPermit

- `publicIp`
- `intranetIp`
- `port`
- `resourceId`
- `username/password`

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
