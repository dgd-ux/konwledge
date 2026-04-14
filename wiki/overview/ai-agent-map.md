# AI Agent Knowledge Map

## Core mental model

An AI agent system in this vault can be read as four layers:

1. Model layer
   - reasoning and generation
2. Memory and context layer
   - short-term context, retrieval, long-term notes
3. Tool and execution layer
   - APIs, browser control, coding tools, test runners
4. Control layer
   - Harness Engineering, rules, validation, permissions, review

## Main topics in this vault

### Agent

- definition of an autonomous task-oriented system
- four capabilities: perception, planning, action, memory
- relation to RAG and workflow automation

See: [Agent](../concepts/agent.md)

### Harness Engineering

- how to keep agents readable, controllable, verifiable, and recoverable
- emphasizes guardrails over prompt-only discipline

See: [Harness Engineering](../concepts/harness-engineering.md)

### Coding agents

- terminal-native assistants such as Claude Code
- memory files, slash commands, project-level navigation

See: [Claude Code](../tools/claude-code.md)

### Automation and testing

- frameworks like OpenClaw illustrate execution reliability, logging, CI, and retries
- useful as examples of verification-heavy agent workflows

See: [OpenClaw](../tools/openclaw.md)

## Useful distinctions

### Agent vs RAG

- RAG extends knowledge access.
- Agent extends task execution.
- RAG can be one tool inside an agent.

### Agent vs workflow automation

- Workflow automation is rule-first and fixed.
- Agent systems are model-guided and adaptive.
- In production, the best systems often combine both.

### Prompt vs Context vs Harness

- Prompt decides what you ask.
- Context decides what the model sees.
- Harness decides how the whole system is allowed to run.

## Reading path

1. [Agent](../concepts/agent.md)
2. [Harness Engineering](../concepts/harness-engineering.md)
3. [Claude Code](../tools/claude-code.md)
4. [OpenClaw](../tools/openclaw.md)
