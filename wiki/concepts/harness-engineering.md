# Harness Engineering

## Definition

Harness Engineering is the engineering discipline of making agent systems readable, controllable, verifiable, and recoverable.

It focuses less on making the model smarter and more on making the system dependable.

## Three pillars

### Readability

Agents need explicit project-facing documentation such as `AGENTS.md`, stable directory structure, and clear completion criteria.

### Defensive control

Important constraints should be enforced by execution rules, not only by prompts.

Typical controls:

- phased execution
- permission boundaries
- mandatory verification after edits
- loop detection

### Feedback loops

A strong system validates results and feeds failure back into future runs.

Typical loops:

- lint
- tests
- typecheck
- review
- lessons learned

## Prompt vs Context vs Harness

- Prompt: how the task is asked
- Context: what the model can see
- Harness: how the system governs action

## Why it matters

Many agent failures are not reasoning failures alone. They are failures of environment design:

- premature claims of completion
- repeated failed actions
- context drift on long tasks
- rule violations under pressure

## Minimal practical implementation

For a small team or personal setup, a minimal harness can already be useful:

1. maintain `AGENTS.md`
2. require verification after edits
3. separate execution from review
4. record recurring mistakes
5. keep handoff notes for long tasks

## Relevance to this vault

This knowledge base itself uses a lightweight harness:

- `raw/` is immutable
- `wiki/` stores durable synthesis
- `index.md` and `log.md` keep structure and history visible
- `AGENTS.md` constrains future maintenance

## Related pages

- [Personal Knowledge Base Practice](../overview/personal-knowledge-base-practice.md)
- [Agent](./agent.md)
- [CSDN Harness Engineering Practice](../sources/csdn-harness-engineering-practice.md)

## Sources

- [CSDN Harness Engineering Practice](../sources/csdn-harness-engineering-practice.md)
