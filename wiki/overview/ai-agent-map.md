# AI Agent 知识地图

## 核心心智模型

在这个知识库里，一个 AI Agent 系统可以拆成四层：

1. 模型层
   - 负责推理与生成
2. 记忆与上下文层
   - 负责短期上下文、检索、长期知识
3. 工具与执行层
   - 负责 API、浏览器、命令行、测试工具等外部动作
4. 控制层
   - 负责 Harness Engineering、规则、验证、权限、评审

## 当前知识库的主要主题

### Agent

- 面向任务的自主系统定义
- 四项核心能力：感知、规划、行动、记忆
- 与 RAG、自动化工作流的关系

见：[Agent](../concepts/agent.md)

### Harness Engineering

- 如何让 Agent 可读、可控、可验证、可恢复
- 强调硬约束与反馈回路，而不是只靠 prompt 自觉

见：[Harness Engineering](../concepts/harness-engineering.md)

### 编程型 Agent

- 以 Claude Code 这类终端原生助手为代表
- 关注项目记忆文件、命令复用、项目级导航

见：[Claude Code](../tools/claude-code.md)

### 自动化与测试

- OpenClaw 这类框架体现了执行可靠性、日志、CI、重试机制
- 可以作为理解验证层与执行层的参考样本

见：[OpenClaw](../tools/openclaw.md)

## 几组重要区分

### Agent vs RAG

- RAG 解决知识获取与事实增强。
- Agent 解决任务执行与行动闭环。
- RAG 常常只是 Agent 的一个工具。

### Agent vs 自动化工作流

- 自动化工作流更依赖固定规则。
- Agent 系统更依赖模型驱动和动态决策。
- 生产系统里通常是两者结合，而不是二选一。

### Prompt vs Context vs Harness

- Prompt 决定你如何提问。
- Context 决定模型看见什么。
- Harness 决定整个系统怎么被允许运行。

## 阅读路径

1. [Agent](../concepts/agent.md)
2. [Harness Engineering](../concepts/harness-engineering.md)
3. [Claude Code](../tools/claude-code.md)
4. [OpenClaw](../tools/openclaw.md)
