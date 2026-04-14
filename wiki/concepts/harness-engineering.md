# Harness Engineering

## 定义

Harness Engineering 是一套让 Agent 系统变得可读、可控、可验证、可恢复的工程方法。

它的重点不是让模型更聪明，而是让整个系统更可靠。

## 三根支柱

### 可读性

Agent 需要清晰读取项目规则、目录结构与完成标准，因此需要类似 `AGENTS.md` 这样的显式说明文件。

### 防御性控制

关键约束应当写进执行规则，而不是只写在 prompt 里。

典型控制包括：

- 分阶段执行
- 权限边界
- 修改后强制验证
- 循环失败检测

### 反馈回路

一个稳健系统不仅要验证结果，还要把失败经验反馈到后续运行中。

典型反馈包括：

- lint
- test
- typecheck
- review
- lessons learned

## Prompt、Context、Harness 的区别

- Prompt：你怎么提问
- Context：模型能看见什么
- Harness：系统如何约束和治理行动

## 为什么重要

很多 Agent 失败并不只是推理能力不足，而是运行环境设计失败：

- 提前宣称完成
- 重复执行失败动作
- 长任务中上下文漂移
- 压力下违反规则

## 最小可用实践

对个人或小团队来说，一个最小版 harness 已经很有价值：

1. 维护 `AGENTS.md`
2. 修改后强制验证
3. 将执行与评审拆开
4. 记录重复错误
5. 为长任务保留交接摘要

## 与本知识库的关系

这个知识库本身就使用了一个轻量 harness：

- `raw/` 不动
- `wiki/` 负责长期沉淀
- `index.md` 和 `log.md` 让结构与历史可见
- `AGENTS.md` 约束未来维护行为

## 相关页面

- [个人知识库实践](../overview/personal-knowledge-base-practice.md)
- [Agent](./agent.md)
- [CSDN Harness Engineering 实践](../sources/csdn-harness-engineering-practice.md)

## 来源

- [CSDN Harness Engineering 实践](../sources/csdn-harness-engineering-practice.md)
