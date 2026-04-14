# runner 执行流程

runner 的职责是接收 system 下发的 `DownloadTask`，通过代理下载资源，走资源处理链，并把结果回调回 system。

## 启动阶段

入口是 `DownloadRunnerApplication`，初始化主要在 `runner` 模块的 `SpringClientContext.run()`。

启动时会做这些事：

- 加载 `AbilityConfig`
- 装配 `ResourceFactory`
- 构造 `RunnerParam`
- 调 system 侧 `/runner/register` 完成注册
- 定时调 `/runner/beat` 发心跳
- 启动时调 `/runner/redistribute` 把残留任务重入队

## system 如何识别 runner 版本

入口是 `RunnerController.switchService()`：

- `runnerParam.runnerVersion == 2` 时走 `RunnerServiceV2`
- 否则走 `DefaultRunnerService`

## 执行主链

1. system 调 runner 侧接口 `POST /task/batch-download`
2. 请求进入 `DownloadController.receiveBatchTask()`
3. 再进入 `RunnerBizImpl.download()`
4. 最终由 `SharedDownloader / DefaultDownloader.download()` 执行

## runner 内部做的事

- 根据 `DownloadTask` 初始化 `Metadata`
- 用 `RequestFactory` 构造 HTTP 请求
- 使用 `ProxyPermit` 发起代理请求
- 下载响应体
- 按能力映射和 MIME 类型选择资源处理链
- 落文件、转码、截图或上传对象存储
- 调 `CompleteCallback` 触发回调

## 来源

- [Source 002: 下载任务调度与 runner 执行链路](../../raw/sources/source-002-download-dispatch-and-runner-flow.md)
