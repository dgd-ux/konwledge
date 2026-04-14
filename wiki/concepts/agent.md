# Agent

## Definition

An agent is an AI system that can interpret goals, plan actions, use tools, and carry work forward with some degree of autonomy.

Across the current sources, the most stable capability model is:

- perception
- planning
- action
- memory

## Core components

### Model

Provides language understanding, reasoning, decomposition, and decision support.

### Tools

Connect the model to external systems such as APIs, browsers, shells, test frameworks, and data stores.

### Memory

- short-term memory: current task state and active context
- long-term memory: reusable preferences, facts, and lessons

### Planning loop

The typical loop is:

`understand -> plan -> act -> observe -> adjust`

## Agent vs plain chatbot

A chatbot mainly answers.

An agent is expected to:

- break a task into steps
- choose tools
- execute against the environment
- use feedback to continue or recover

## Agent vs RAG

- RAG improves factual grounding.
- Agent improves task completion.
- RAG is often one subsystem of an agent rather than a replacement for it.

## Agent vs workflow automation

- Automation works best for deterministic branches.
- Agents work best for ambiguous, changing, or multi-step work.
- Practical systems often use deterministic workflows to constrain agent execution.

## Practical implications

- Strong agent behavior is not only about the model.
- Good tool design and control layers matter as much as raw intelligence.
- Personal knowledge bases can serve as long-term memory for learning and implementation.

## Related pages

- [AI Agent Knowledge Map](../overview/ai-agent-map.md)
- [Harness Engineering](./harness-engineering.md)
- [Claude Code](../tools/claude-code.md)
- [Aliyun Agent Explained](../sources/aliyun-agent-explained.md)
- [CSDN What Is Agent](../sources/csdn-what-is-agent.md)

## Sources

- [Aliyun Agent Explained](../sources/aliyun-agent-explained.md)
- [CSDN What Is Agent](../sources/csdn-what-is-agent.md)
