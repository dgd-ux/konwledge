# 个人知识库实践

## 目标

建立一套既适合人阅读，也适合 LLM 持续维护与查询的个人知识库。

本仓库当前采用一个简单但稳定的结构：

- `raw/` 存放不改动的原始材料
- `wiki/` 存放可复用的提炼结果
- `AGENTS.md` 约束未来如何继续维护

## 为什么这套结构有效

- 原始材料完整保留，后续可以反复重读和重新解释。
- `wiki/` 层把零散阅读转成长期可复用知识。
- `schema` 层避免知识库退化成一堆无规则堆放的 markdown。

## 推荐工作流

### 1. 收集

把裁剪文章、访谈记录、PDF 转 markdown、网页摘录和临时资料放进 `raw/`。

### 2. 提炼

对每份重要材料：

- 在 `wiki/sources/` 创建来源页
- 把稳定概念抽取到概念页
- 把工具或框架抽取到工具页

### 3. 综合

当多份材料指向同一主题时，把长期有效的结论汇总到：

- `wiki/concepts/`
- `wiki/tools/`
- `wiki/overview/`

### 4. 复用

后续提问时优先查 `wiki/`，只有在需要细节或核实时再返回 `raw/`。

### 5. 维护

随着知识库增长：

- 更新 [index.md](../../index.md)
- 向 [log.md](../../log.md) 追加维护记录
- 优先补充已有主题页，而不是生成大量重复页

## 当前主题簇

- Agent 基础概念与能力模型
- Harness Engineering 与控制层
- AI 编程助手
- 自动化与测试实践

## 实用笔记模式

面对新材料，优先按这个顺序处理：

1. 阅读并摘要来源。
2. 判断这份来源更偏概念、战术还是宣传。
3. 把它挂到现有主题页上。
4. 至少写下一条长期有效结论和一条待验证问题。

## 应避免的情况

- 在整理阶段直接修改 `raw/`
- 只有文章摘要，没有主题沉淀
- 多个页面反复用不同措辞写同一件事
- 稳定知识和临时判断混在一起却不标注

## 下一步值得补充的内容

- 增加 `RAG`、`MCP`、`Context Engineering`、`multi-agent review` 页面
- 如果后续开始跟踪具体实践项目，可以增加 `projects/`
- 如果想维护长期问题清单，可以增加 `questions/`

## 相关页面

- [AI Agent 知识地图](./ai-agent-map.md)
- [Agent](../concepts/agent.md)
- [Harness Engineering](../concepts/harness-engineering.md)
- [Claude Code](../tools/claude-code.md)
