# Agent

## 定义

Agent 是一种能够理解目标、规划步骤、调用工具并在一定自主性下推进任务的 AI 系统。

结合当前知识库里的材料，最稳定的能力模型是四项：

- 感知
- 规划
- 行动
- 记忆

## 核心组成

### 模型

负责语言理解、推理、任务拆解与决策支持。

### 工具

把模型连接到外部世界，例如 API、浏览器、命令行、测试框架和数据源。

### 记忆

- 短期记忆：保存当前任务状态和活跃上下文
- 长期记忆：保存可复用偏好、事实和经验

### 规划闭环

典型工作链路可以写成：

`理解 -> 规划 -> 执行 -> 观察 -> 调整`

## Agent 与普通聊天模型的区别

普通聊天模型主要负责回答问题。

而 Agent 需要进一步做到：

- 将任务拆成步骤
- 选择和调度工具
- 对环境执行动作
- 根据反馈继续推进或修正

## Agent 与 RAG 的关系

- RAG 更偏知识增强与事实补充。
- Agent 更偏任务执行与行动闭环。
- 在很多系统里，RAG 只是 Agent 的一个子能力，而不是替代品。

## Agent 与自动化工作流的关系

- 自动化工作流更适合确定性分支。
- Agent 更适合模糊、变化、多步骤任务。
- 实际工程里，常常用固定工作流来约束 Agent 的执行边界。

## 实践含义

- Agent 的效果不只取决于模型本身。
- 工具设计、上下文设计和控制层设计同样关键。
- 个人知识库可以承担 Agent 的长期记忆载体角色。

## 相关页面

- [AI Agent 知识地图](../overview/ai-agent-map.md)
- [Harness Engineering](./harness-engineering.md)
- [Claude Code](../tools/claude-code.md)
- [阿里云 Agent 综述](../sources/aliyun-agent-explained.md)
- [CSDN 什么是 Agent](../sources/csdn-what-is-agent.md)

## 来源

- [阿里云 Agent 综述](../sources/aliyun-agent-explained.md)
- [CSDN 什么是 Agent](../sources/csdn-what-is-agent.md)
