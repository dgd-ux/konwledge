niop-open-download 任务调度核心流程说明
本文不按类清单展开，而是按“一个下载任务进入系统后，如何被拉取、调度、执行、回调”的时间顺序来讲。
重点回答三个问题：
1. 任务从哪里进来。
2. 什么时候走 v1，什么时候走 v2。
3. runner 执行完之后，结果如何回写并结束任务。
1. 先看整体：项目里其实有两套调度链路
   系统启动后会同时启动下面几类组件：
   ● TaskPullerV1：v1 拉取链路。
   ● TaskPuller：v2 拉取链路。
   ● DistributorV1：v1 调度器。
   ● DownloadTaskExecutor：v2 执行器。
   ● DistributeSchedule -> LiveDistributor：直播场景的 v2 调度入口。
   也就是说，这个项目不是“完全从 v1 升级到 v2”，而是“v1、v2 并存”。
2. 一个任务是怎么进系统的
   2.1 外部主动提交
   外部接口在 TaskController：
   ● POST /task/live/submit
   ● POST /task/live/batch-submit
   这里会进入 TaskService.submit/submitBatch，先做几件事：
   ● 根据 appId + abilityId 找到 AppConfig。
   ● 根据 abilityId 找到 Ability。
   ● 根据 mediaId + groupId 计算当前资源池可承载的最大并发量。
   ● 如果当前未完成任务数已经超过上限，直接拒绝提交。
   然后进入 AtspTaskProcessor.submitTask/submitTasks：
   ● 先把任务提交到 ATSP。
   ● 再把任务转成系统内部的 DistributeTask。
   ● 最后写入任务表。
   这里有一个关键点：
   ● AtspTaskProcessor.buildTask() 会把 DistributeTask.distributeVersion 直接设为 2。
   这说明“新提交流程”默认是按 v2 任务对象在落库。
   2.2 从 ATSP 被动拉取
   系统自己也会持续从 ATSP 拉任务，这里分成两套：
   ● TaskPullerV1：老链路拉取。
   ● TaskPuller：新链路拉取。
   这也是后面 v1/v2 分流的根本来源。
3. 什么时候走 v1，什么时候走 v2
   3.1 走 v1 的条件
   从代码结构看，v1 仍然是默认的大盘链路，特点是：
   ● TaskPullerV1 启动后，按“资源类型 mediaId”维度去 ATSP 拉任务。
   ● 拉下来的任务会进入 DistributeTaskDao 的 Redis 队列，队列 key 是 csp + mediaId。
   ● 后续由 DistributorV1 持续消费这些队列。
   也就是说，下面这些情况会走 v1：
   ● 非直播场景。
   ● 不在 v2 固定媒体集合中的任务。
   ● 仍然依赖“按云厂商 + 资源类型分队列”方式调度的任务。
   v2 固定媒体集合在 Constants.V2_DISTRIBUTE_MEDIAS：
   ● dy-picture
   ● dy-video
   ● xhs-picture
   ● xhs-video
   因此，从代码意图上看：
   ● 不属于上面 4 个媒体类型的普通下载任务，仍然主要走 v1。
   3.2 走 v2 的条件
   v2 目前不是覆盖全量，而是覆盖两类场景：
1. 固定媒体类型场景
2. 直播能力场景
   固定媒体类型场景
   DownloadTaskExecutor.start() 只会启动这些 ability 的调度任务：
   ● ability.mediaId 在 Constants.V2_DISTRIBUTE_MEDIAS 中
   也就是：
   ● 抖音图片
   ● 抖音视频
   ● 小红书图片
   ● 小红书视频
   这类任务会由 TaskPuller 拉取后，进入 TaskDao 对应表，再由 DownloadTaskExecutor 定时调用 DistributorV2.distribute(task)。
   直播能力场景
   直播没有走 DownloadTaskExecutor 这条普通 v2 执行器，而是单独走：
   ● DistributeSchedule.labelDispatch()
   ● LiveDistributor.distribute()
   直播能力 ID 目前硬编码在 LiveDistributor.LIVE_ABILITY：
   ● 51QXQ1VQ
   ● GOIWISWT
   ● PRENV2PQ
   也就是说：
   ● 直播任务也是 v2 思路，但入口是单独的定时调度器，不和普通 v2 共用一个执行器。
   3.3 一个容易误解的点
   虽然很多新任务在入库时已经被写成 distributeVersion=2，但“真正走哪套调度器”不只看这个字段，还看：
   ● 任务是不是由 TaskPullerV1 还是 TaskPuller 拉进来。
   ● ability.mediaId 是否属于 V2_DISTRIBUTE_MEDIAS。
   ● abilityId 是否属于直播 LIVE_ABILITY。
   换句话说：
   ● distributeVersion 更像结果标记和回调分流标记。
   ● 真正的执行入口，还是由不同的 puller / distributor / executor 决定。
4. v1 主流程
   4.1 拉任务
   入口：TaskPullerV1.start()
   核心动作：
   ● 先从 MediaWithAbilitiesService 取出“资源类型和能力”的绑定关系。
   ● 按 mediaId 判断每类资源当前队列长度。
   ● 如果某类资源队列未满，则调用 AtspClient.batchPullTasks(abilityId, expectedTaskSize) 拉任务。
   ● 每个任务按 appId 找 AppConfig，再拿到对应 csp 和 groupId。
   ● 转成 DistributeTask 后写入：
   ○ MySQL 任务表
   ○ Redis 队列 DistributeTaskDao
   这条链路的核心特征是：
   ● 任务先进入“按资源类型分桶”的队列。
   ● 后续调度是先选资源，再取任务，而不是逐任务直接选 runner。
   4.2 调度
   入口：DistributorV1.start()
   它会为每个 Dispatcher 起一个线程。当前注册在 DispatcherRegistry 里的主要是：
   ● AliyunDispatcher
   ● HwcDispatcher
   每个 Dispatcher 的核心流程都在 BaseDispatcher.doDispatch()：
1. 先选 runner。
2. 再选一个资源类型 mediaId。
3. 看这个资源类型在当前云厂商下有没有任务。
4. 按任务数尽可能多地申请代理 permit。
5. 从 Redis 队列里批量取任务。
6. 组装成 DownloadTask 列表。
7. 批量调用 runner 的 task/batch-download。
8. 成功的任务改为 PROCESS，失败的任务和代理重新入队。
   这里 v1 的设计重点是“批量”：
   ● 一个 runner 一次可以拿到一批任务。
   ● 一个资源类型会被成批处理。
   ● 调度中心更像“资源池分发器”。
   4.3 v1 关键状态变化
   任务成功下发后：
   ● DefaultRunnerService.useRunner() 增加 runner 的负载计数。
   ● RunnerTaskDao.cacheTasks() 记录“这个 runner 正在执行哪些任务”。
   ● DistributeTask.state = PROCESS
   ● DistributeTask.distributeTime = now
   如果下发失败：
   ● 已申请的代理 permit 释放。
   ● 失败任务通过 DistributeTaskDao.rejoinQueue() 回到队列头部。
5. v2 主流程
   5.1 拉任务
   入口：TaskPuller.start()
   它不是按资源类型拉，而是按“当前节点负责的 ability”拉：
1. 根据 Nacos 和当前节点能力，得到本节点负责的 Ability 列表。
2. 对每个 ability 定时执行 AbilityTaskLoader。
3. 调 serverClient.getConsumerClient().batchPull(abilityId, expectedSize) 拉任务。
4. 通过 AtspTaskProcessor.parse() 转成 DistributeTask。
5. 批量写入 TaskDao.batchSave()。
   这里生成的任务对象会显式带上：
   ● distributeVersion = 2
   v2 的核心特征是：
   ● 不再先放入“按资源类型分桶”的 Redis 队列。
   ● 而是直接把任务存到任务表，后面按 task 粒度调度。
   插入任务的APPConfig：根据这个任务可以拼出ConfigId(AppId+AbilityId，是唯一的)，然后根据ConfigId查询出AppConfig（niop_open_download_app_config和niop_open_download_biz_group连表查询），设置csp、configId、groupId...
   5.2 普通 v2 调度
   入口：DownloadTaskExecutor.start()
   它只处理 mediaId 属于 V2_DISTRIBUTE_MEDIAS 的能力。
   每轮执行：
1. taskDao.fetchTasks(abilityId) 拉出该 ability 当前待处理任务。
2. 对每个任务调用 DistributorV2.distribute(task)。
   DistributorV2 当前 handler 链非常短：
1. LoadBalancedSelectHandler
2. SendTaskHandler
   LoadBalancedSelectHandler 做了什么
   它是 v2 的关键：
   ● 先对 groupId 加分布式锁。
   ● 通过 groupId 获取到改groupId所有的runner（Redis），从小根堆选负载最低 runner。(从缓存查，没有缓存则通过groupId查库)
   ● 通过 PermitService.tryAcquireRunner() 原子增加 runner 负载。
   ● 再通过 PermitService.peekProxy() 选负载最低 proxy。
   ● 通过 PermitService.tryAcquireProxy() 原子增加 proxy 负载。
   ● 再用 PermitController.allowAccess() 检查代理对当前媒体的 QPS 限流。
   成功后，任务上下文里会得到：
   ● selectedRunner
   ● selectedProxy
   SendTaskHandler 做了什么
   它负责把 DistributeTask 真正转成 runner 能执行的 DownloadTask：
   根据上一步获取到的Runner填充下载地址（RunnerBizClient）：http://%s/open/download/runner/
   ● 填充 taskId/appId/abilityId/configId
   ● 填充下载 URL、headers、duration
   ● 生成上传路径 path
   ● 写入 proxyPermit
   ● 设置 downloadTask.distributeVersion = 2
   然后调用：
   ● RunnerBizClient.download()
   ● 对应 HTTP 接口是 runner 侧 POST /task/batch-download
   如果下发成功：
   ● TaskDao.runningState() 更新任务状态为 PROCESS
   ● 同时写入 runner_task 表
   ● 同时增加 runner/proxy 当前 permit
   如果下发失败：
   ● 释放代理 permit
   ● 回滚 runner/proxy 的负载占用
   5.3 直播 v2 调度
   入口：DistributeSchedule.labelDispatch() -> LiveDistributor.distribute()
   直播链路没有用 LoadBalancedSelectHandler，而是走一条兼容旧逻辑的 handler 链：
1. MatchRunnersHandler
2. MatchProxiesHandler
3. SelectRunnerHandler
4. SelectProxyHandler
5. SendTaskHandler
   区别在于：
   ● 普通 v2：直接从 Redis 小根堆选最小负载。
   ● 直播 v2：先查全量 runner/proxy，再按当前负载和 permit 逐步选择。
   所以直播虽然也算 v2，但它不是“普通 v2 负载堆模型”，而是一个过渡态实现。
6. runner 侧执行流程
   runner 启动入口是 DownloadRunnerApplication，实际初始化在 runner 模块的 SpringClientContext.run()。
   启动时会做这些事：
   ● 加载能力配置 AbilityConfig
   ● 装配资源处理链 ResourceFactory
   ● 构造 RunnerParam
   ● 调 system 侧 /runner/register 完成注册
   ● 定时调 /runner/beat 发心跳
   ● 启动时调 /runner/redistribute，把自己残留任务重新入队
   6.1 system 如何区分 runner 走 v1 还是 v2
   入口：RunnerController
   RunnerController.switchService() 的规则很简单：
   ● runnerParam.runnerVersion == 2 -> RunnerServiceV2
   ● 否则 -> DefaultRunnerService（v1）
   也就是说：
   ● runner 的注册、心跳、下线、任务重入队，是靠 RunnerParam.runnerVersion 分流的。
   6.2 runner 如何执行任务
   system 下发到 runner 后，请求进入：
   ● DownloadController.receiveBatchTask()
   ● RunnerBizImpl.download()
   ● SharedDownloader / DefaultDownloader.download()
   实际上最后是走的DefaultDownloader，直接看DefaultDownloader
   DefaultDownloader 的主流程：
1. 根据 DownloadTask 初始化 Metadata。
2. 用 RequestFactory 构造 HTTP 请求。
3. 使用 ProxyPermit 建立代理请求。
4. 下载响应体。
5. 根据能力映射和 MIME 类型，选择资源处理链。
6. 资源处理链负责：
   ○ 落文件
   ○ 转码
   ○ 截图
   ○ HLS/FLV 处理
   ○ 上传对象存储
7. 成功或失败后触发 CompleteCallback。
7. 回调主流程
   runner 侧回调入口是 DefaultCompleteCallback.postProcess()：
   ● 组装 ResultParam
   ● 调 system 侧 POST /callback
   system 侧回调入口：
   ● CallbackController.receive()
   ● ServerBizImpl.callback()
   ServerBizImpl 会先看：
   ● ResultParam.distributeVersion
   规则如下：
   ● distributeVersion == 2 -> CallbackServiceV2.receive()
   ● 否则 -> CallbackServiceV1.receive()
   这就是回调阶段真正的 v1/v2 分流点。
   7.1 v1 回调
   CallbackServiceV1 的特点：
   ● 非火山海外云会先释放代理 permit。
   ● 成功时如果业务码是下载错误，并且重试次数没超过 3 次，会把任务重新入队。
   ● 最终会：
   ○ 回调 ATSP
   ○ 保存下载结果
   ○ 从 runner_task 队列移除任务
   ○ 更新任务状态为 SUCCESS 或 FAILED
   7.2 v2 回调
   CallbackServiceV2 的特点：
   ● 先 taskDao.remove(taskId)，移除 runner_task 映射。
   ● 同时减少 runner/proxy 当前 permit。
   ● 释放代理 permit。
   ● 对 V2_DISTRIBUTE_MEDIAS 中的任务，还会调用 PermitService.releaseAllPermits() 释放负载堆中的 runner/proxy 占用。
   ● 如果业务码是错误且重试次数小于 3：
   ○ 只把任务改回 WAIT
   ○ retryTimes + 1
   ● 否则：
   ○ 保存结果
   ○ 回调 ATSP
   ○ 更新状态为 SUCCESS 或 FAILED
   v2 的核心是：
   ● permit、当前负载、负载堆状态，要一起回收。
   ● 它比 v1 多了一层“负载均衡状态回收”。
8. 关键对象，只看最重要的字段
   8.1 DistributeTask
   这是 system 侧的核心任务对象，字段里最值得盯的是：
   ● taskId：任务唯一标识。
   ● abilityId：决定下载能力。
   ● appId：决定应用配置。
   ● configId：决定上传配置、groupId 等。
   ● mediaId：资源类型，决定规则、permit、v1/v2覆盖范围。
   ● groupId：决定 runner/proxy 资源池。
   ● csp：云厂商。
   ● state：WAIT / PROCESS / SUCCESS / FAILED。
   ● retryTimes：回调失败后是否重试。
   ● distributeVersion：回调时决定走 CallbackServiceV1 还是 CallbackServiceV2。
   ● distributeTime / finishTime / executionTimeMs：执行时长统计关键字段。
   8.2 DownloadTask
   这是 system 发给 runner 的执行对象，关键字段：
   ● id：任务 ID。
   ● abilityId
   ● appId
   ● configId
   ● url
   ● headers
   ● duration
   ● path：上传目标路径。
   ● proxyPermit：runner 下载时真正使用的代理。
   ● retryTimes
   ● distributeVersion
   8.3 ResultParam
   这是 runner 回调给 system 的对象，关键字段：
   ● taskId
   ● abilityId
   ● resultView：最终业务结果。
   ● proxyPermit
   ● runnerIp
   ● retryTimes
   ● downloadRecord
   ● distributeVersion
   8.4 RunnerParam
   这是 runner 注册/心跳对象，关键字段：
   ● ip
   ● port
   ● csp
   ● taskAmount
   ● hostName
   ● runnerVersion
   其中最关键的是：
   ● runnerVersion：system 用它决定 runner 管理接口走 v1 还是 v2。
   8.5 RunnerDTO
   这是 v2 选择 runner 时的重要对象，关键字段：
   ● address：system 调 runner HTTP 接口时使用。
   ● ip
   ● maxPermits
   ● currentPermits
   ● state
   ● csp
   8.6 ProxyPermit
   这是 system 发给 runner 的代理凭证，关键字段：
   ● publicIp
   ● intranetIp
   ● port
   ● resourceId
   ● username/password
   它既是 runner 下载代理配置，也是 system 回调后释放 permit 的依据。
9. 最后用一句话总结
   这个项目的真实结构不是“单一下载链路”，而是：
   ● v1 负责大部分传统任务，核心是“按资源类型分队列，批量调度”。
   ● v2 负责一部分新媒体和直播能力，核心是“按任务逐个调度，按 groupId 做 runner/proxy 负载均衡”。
   ● runner 执行链路基本共用，真正分流发生在“任务拉取”和“回调处理”两个地方。
   如果你继续往下看代码，建议优先盯下面这几条链：
   ● 提交链：TaskController -> TaskService -> AtspTaskProcessor
   ● v1：TaskPullerV1 -> DistributeTaskDao -> DistributorV1 -> BaseDispatcher
   ● 普通 v2：TaskPuller -> DownloadTaskExecutor -> DistributorV2 -> LoadBalancedSelectHandler -> SendTaskHandler
   ● 直播 v2：DistributeSchedule -> LiveDistributor
   ● runner：DownloadController -> RunnerBizImpl -> DefaultDownloader -> DefaultCompleteCallback
   ● 回调：CallbackController -> ServerBizImpl -> CallbackServiceV1/CallbackServiceV2












通过指定代理发起一次 HTTP 请求，拿到响应后做统一校验、类型识别、策略匹配，然后把响应体交给对应的处理链去解析。
可以把整条链路理解成 6 个阶段：
1. 构造带代理的 HTTP 调用
2. 执行请求并拿到响应
3. 根据响应码判断资源状态
4. 校验响应体和内容大小
5. 根据能力 ID 和 MIME 类型选择处理策略
6. 把响应体交给责任链处理
   下面按流程详细拆。

1. 方法整体职责
   方法签名：
   private ResourceState execute(Metadata metadata, Request request, ProxyPermit proxy) throws IOException
   入参含义大致是：
   ● metadata：贯穿全流程的上下文，保存下载/解析过程中的状态、失败原因、文件大小等信息
   ● request：OkHttp 的请求对象
   ● proxy：代理配置，包含代理 IP、端口、认证信息
   返回值：
   ● ResourceState：资源处理结果状态，比如成功、失败、超大、非法等

2. 第一步：构造带代理认证的 Call
   Call call = client.newBuilder()
   .proxyAuthenticator((route, response) -> response.request().newBuilder()
   .header(AUTHORIZATION_HEADER, proxy.getAuthorization()).build())
   .proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxy.getIntranetIp(), proxy.getPort())))
   .build()
   .newCall(request);
   这一段是在基于已有的 client 创建一个新的 OkHttpClient，并给它附加当前请求专属的代理设置。
   这里做了两件事
   2.1 设置代理认证器 proxyAuthenticator
   .proxyAuthenticator((route, response) -> response.request().newBuilder()
   .header(AUTHORIZATION_HEADER, proxy.getAuthorization()).build())
   作用是：
   ● 当代理服务器要求认证时，OkHttp 会回调这里
   ● 然后重新构造请求
   ● 给请求头加上代理认证信息，比如 Proxy-Authorization
   也就是说，这不是给目标网站认证，而是给代理服务器认证。

2.2 设置代理地址
.proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxy.getIntranetIp(), proxy.getPort())))
作用是：
● 指定当前请求走 HTTP 代理
● 代理地址来自 proxy 对象中的内网 IP 和端口
所以请求链路就变成：
当前服务 -> 代理服务器 -> 目标资源服务器

3. 第二步：执行请求
   try (final Response response = call.execute()) {
   这里是同步执行 HTTP 请求。
   特点：
   ● execute() 是阻塞调用
   ● 一直到拿到响应或者抛异常才返回
   ● 用 try-with-resources 自动关闭 response
   如果网络异常、超时、连接失败等，会直接抛 IOException，由方法继续向上抛。

4. 第三步：根据响应码判断资源状态
   final ResourceState resourceState = ResourceState.getState(response.code());
   metadata.setAttribute(Key.Resource.STATE, resourceState);
   这里做了两件事：
   ● 用 HTTP 状态码映射出系统内部定义的 ResourceState
   ● 把这个状态写入 metadata
   例如可能类似：
   ● 200 -> SUCCESS
   ● 404 -> NOT_FOUND
   ● 403 -> FORBIDDEN
   ● 500 -> FAIL

如果响应状态失败，直接返回
if (resourceState.isFail()) {
metadata.setAttribute(Key.Record.FAIL_REASON, resourceState.getMessage() + ", code: " + response.code());
return resourceState;
}
这里体现了一个很重要的流程控制思想：
先看“资源是否可访问”，访问都失败了，就没必要继续读 body 和解析内容。
所以失败链路是：
● 根据 response.code() 得到失败状态
● 记录失败原因到 metadata
● 立即返回
这属于快速失败。

5. 第四步：读取并校验响应体
   try (final ResponseBody body = response.body()) {
   这里进入响应体处理阶段。
   注意：
   ● response 和 body 都用了 try-with-resources
   ● 表示方法结束时会自动释放网络资源和流资源

5.1 判空
if (body == null) {
metadata.setAttribute(Key.Record.FAIL_REASON, "response body is null");
return resourceState;
}
即使 HTTP 响应码成功，也不代表一定有 body。
比如：
● 某些响应本身没有实体
● 或上游返回异常格式
如果 body 为空：
● 记录失败原因
● 返回当前 resourceState
这里返回的还是原状态，而不是新造一个失败状态，说明设计上更强调： HTTP 访问状态是成功的，但内容不可用。

5.2 获取 MIME 类型
final MimeType originMediaType = MimeType.getMediaType(body.contentType());
作用：
● 从响应头 Content-Type 中解析出资源原始类型
● 比如：
○ text/html
○ application/pdf
○ image/jpeg
这个类型后面用于决定该资源该怎么解析。

5.3 获取文件大小
final long contentLength = body.contentLength();
metadata.setAttribute(Key.File.SIZE, contentLength);
作用：
● 读取响应体声明的长度
● 保存到 metadata
注意这里的 contentLength() 可能是：
● 正常值
● 或 -1，表示服务端没提供长度

5.4 做大小限制校验
if (contentLength >= MAX_CONTENT_LENGTH) {
metadata.setAttribute(Key.Record.FAIL_REASON, "content length is greater than MAX_CONTENT_LENGTH");
return ResourceState.OVERSIZE;
}
这里是一个保护机制：
● 如果文件太大，不继续处理
● 直接返回 OVERSIZE
目的通常是：
● 防止大文件拖垮内存/磁盘/解析器
● 限制单任务资源消耗
这里要注意一点：
如果 contentLength == -1，通常不会命中这个条件，因为 -1 >= MAX_CONTENT_LENGTH 一般为 false。

6. 第五步：根据能力 ID 和类型选择处理策略
   final String abilityId = metadata.getAttribute(Key.Task.ABILITY_ID);
   final MimeType inboundMediaType = abilityMediaMapping.getInboundMediaType(abilityId, originMediaType);
   这一步开始进入“业务解析层”。
   6.1 获取能力 ID
   abilityId 可以理解成：
   ● 当前任务属于哪种业务能力
   ● 不同能力对资源类型支持不同
   ● 同一个原始类型在不同能力下处理方式也可能不同
   比如：
   ● 网页抽取能力
   ● 文档解析能力
   ● 图片识别能力

6.2 计算输入到处理链的媒体类型
final MimeType inboundMediaType = abilityMediaMapping.getInboundMediaType(abilityId, originMediaType);
这里不是直接拿原始 MIME 去处理，而是做了一次映射。
说明系统里有一层“能力类型适配”：
● 原始资源类型 originMediaType
● 在某个 abilityId 下，被转换/归一成某种 inboundMediaType
例如：
● 原始是 application/octet-stream
● 但某能力下可能被归类成 PDF 或二进制文档流
● 或者 text/plain / text/html 被统一映射成某个可解析类型

7. 第六步：解析处理策略
   final HandlerStrategy strategy = resolveHandlerStrategy(abilityId, originMediaType, inboundMediaType);
   这一步很关键。
   strategy 里至少包含：
   ● targetType
   ● chain
   ● inboundType
   从命名看，它是一个“处理策略对象”，决定：
   ● 最终要转成什么目标类型
   ● 用哪条处理链
   ● 处理链接收什么输入类型
   可以理解成：
   前面是在判断“能不能处理”，这里是在决定“具体怎么处理”。

如果没有匹配到策略，直接返回非法
if (strategy == null) {
metadata.setAttribute(Key.Record.FAIL_REASON, "能力不支持解析该类型的资源,类型: " + originMediaType);
return ResourceState.ILLEGAL;
}
说明：
● 当前能力不支持这个资源类型
● 或者系统没有为这个类型配置处理链
这是业务上的“不合法资源”。

8. 第七步：创建执行上下文
   final DefaultExecutionContext context = new DefaultExecutionContext(metadata, strategy.targetType, body, resourceState, proxy);
   这里把后续处理所需的上下文统一封装起来。
   上下文里包含：
   ● metadata：全流程共享元数据
   ● strategy.targetType：目标输出类型
   ● body：响应体数据流
   ● resourceState：当前资源状态
   ● proxy：代理信息
   这一步的意义是：
   把调用责任从当前方法转移给处理链，同时把它所需的一切数据准备好。

9. 第八步：责任链执行真正解析
   strategy.chain.doHandle(context, strategy.inboundType);
   这是整段代码的最终落点。
   前面的代码基本都在做准备工作，真正的“内容解析”是在这里发生的。

这里的设计模式很明显是“责任链 + 策略”
策略模式
通过 resolveHandlerStrategy(...) 选择不同处理策略：
● 不同能力
● 不同原始类型
● 不同入站类型
● 对应不同处理链
责任链模式
strategy.chain.doHandle(...) 表示交给一个链式处理器去逐步处理。
责任链里可能包含：
● 解压
● 转码
● 格式识别
● 文本提取
● 清洗
● 落库
● 结果回写 metadata
当前方法本身不关心链内部细节，它只负责：
● 发请求
● 做入口校验
● 选中正确的链
● 把执行权交出去
这就是典型的调度层 / orchestration 层。

整体流程图
可以把链路抽象成这样：
入参(metadata, request, proxy)
->
构造带代理和认证的 OkHttp Call
->
执行 HTTP 请求
->
根据响应码得到 ResourceState
->
失败? 是 -> 记录失败原因并返回
->
读取 ResponseBody
->
body 为空? 是 -> 记录失败原因并返回
->
解析 Content-Type 和 Content-Length
->
文件过大? 是 -> 返回 OVERSIZE
->
从 metadata 取 abilityId
->
根据 abilityId + originMediaType 映射 inboundMediaType
->
解析 HandlerStrategy
->
策略不存在? 是 -> 返回 ILLEGAL
->
构造 DefaultExecutionContext
->
交给 strategy.chain.doHandle(...) 执行解析
->
返回 resourceState

这段代码在系统中的角色
如果从分层角度看，这段代码像是一个资源获取 + 解析调度入口，职责边界比较清晰：
它负责的事
● 代理访问资源
● 统一处理 HTTP 层结果
● 做基础校验
● 根据业务能力选择处理链
● 启动解析流程
它不负责的事
● 不负责具体解析实现
● 不负责链内每一步的细节
● 不负责策略本身的定义
所以这段代码更像一个：
“下载执行器 / 资源处理入口 / 责任链分发器”

你重点可以这么理解“链路”
第一段链路：网络访问链路
request + proxy
-> OkHttpClient(代理、认证)
-> call.execute()
-> response
这是“把资源拿回来”。

第二段链路：状态校验链路
response.code()
-> ResourceState
-> 失败直接返回
-> body 判空
-> contentLength 校验
这是“判断资源是否值得继续处理”。

第三段链路：业务路由链路
abilityId + originMediaType
-> inboundMediaType
-> resolveHandlerStrategy(...)
-> 找到对应 chain
这是“决定走哪条解析通道”。

第四段链路：内容处理链路
DefaultExecutionContext
-> strategy.chain.doHandle(...)
这是“真正解析资源内容”。

这段代码的几个设计点
1. 快速失败
   一旦发现：
   ● 响应码失败
   ● body 为空
   ● 内容过大
   ● 类型不支持
   就立即返回，不进入后续链路。
   优点是节省资源，逻辑清晰。

2. 元数据贯穿全流程
   metadata 是整个方法里的核心上下文载体。
   它记录了：
   ● 资源状态
   ● 失败原因
   ● 文件大小
   ● abilityId
   这样后续链路和外层调用方都能共享状态。

3. 网络层和业务层解耦
   这段代码只做“获取资源 + 调度处理”，具体解析交给 strategy.chain。
   这让系统比较容易扩展：
   ● 新增能力
   ● 新增 MIME 类型支持
   ● 新增处理链节点
   都不一定要改这段主流程。

4. 责任链扩展性强
   strategy.chain.doHandle(...) 说明不同资源可以走不同链，链内部还能按节点拆分处理逻辑，适合复杂解析系统。

一个更通俗的比喻
可以把这段代码理解成一个“快递分拣中心”：
1. 先通过代理把包裹取回来
2. 检查包裹状态是否正常
3. 看包裹有没有内容
4. 检查包裹是不是超重
5. 根据业务类型和包裹种类决定送到哪条分拣线
6. 把包裹交给对应流水线处理
   它自己不负责“拆包裹里的每个东西”，只负责把包裹正确地送到正确的处理线。

如果你愿意，我下一步可以继续帮你讲这几个点之一：
1. strategy.chain.doHandle(context, strategy.inboundType) 这一层责任链模式怎么设计
2. abilityId / originMediaType / inboundMediaType 三者的关系
3. 这段代码有哪些潜在风险和可优化点
4. 按时序图方式把整段调用链画出来