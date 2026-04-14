# niop-open-download LLM Wiki

这个目录是基于项目现有知识笔记整理出的本地 LLM wiki。结构参考 Karpathy 提出的 `raw sources + wiki + schema + log` 方式，目标是让后续人类或 agent 能持续增量维护，而不是反复从零总结。

## 入口

- [更新日志](./log.md)
- [维护约定](./AGENTS.md)
- [分类与命名规则](./schema/taxonomy.md)

## 原始资料

- [Source 001: RBAC 权限控制设计](./raw/sources/source-001-rbac-permission-design.md)
- [Source 002: 下载任务调度与 runner 执行链路](./raw/sources/source-002-download-dispatch-and-runner-flow.md)

## Source Notes

- [RBAC 权限设计笔记](./wiki/source-notes/rbac-permission-design-note.md)
- [下载调度链路笔记](./wiki/source-notes/download-dispatch-flow-note.md)

## 权限系统

- [RBAC 权限模型](./wiki/auth/rbac-permission-model.md)
- [权限校验链路](./wiki/auth/permission-validation-flow.md)

## 架构与执行链路

- [调度架构总览](./wiki/architecture/dispatch-architecture-overview.md)
- [v1 与 v2 调度分流](./wiki/architecture/v1-v2-routing.md)
- [runner 执行流程](./wiki/architecture/runner-execution-flow.md)
- [回调与状态回收](./wiki/architecture/callback-and-state-recovery.md)
- [资源处理执行器](./wiki/architecture/resource-processing-executor.md)
- [核心对象速查](./wiki/architecture/core-objects-cheatsheet.md)
