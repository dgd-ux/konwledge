# Personal Knowledge Base Practice

## Goal

Build a personal knowledge base that is easy for both humans and LLMs to read, extend, and query.

This vault uses a simple pattern:

- `raw/` stores untouched materials.
- `wiki/` stores reusable understanding.
- `AGENTS.md` defines how future updates should happen.

## Why this structure works

- Raw material stays intact, so future re-interpretation is possible.
- The wiki layer turns scattered reading into durable notes.
- The schema layer prevents the vault from drifting into random markdown dumps.

## Recommended workflow

### 1. Capture

Put clipped articles, transcripts, PDFs converted to markdown, and temporary notes into `raw/`.

### 2. Distill

For each important raw item:

- create a source note in `wiki/sources/`
- extract stable concepts into concept pages
- extract tools or frameworks into tool pages

### 3. Synthesize

When several sources point to the same theme, merge the durable insight into:

- `wiki/concepts/`
- `wiki/tools/`
- `wiki/overview/`

### 4. Reuse

When asking future questions, query `wiki/` first. Only return to `raw/` when nuance or verification is needed.

### 5. Maintain

Whenever the vault grows:

- update [index.md](../../index.md)
- append to [log.md](../../log.md)
- refine existing topic pages instead of creating duplicates

## Current topic clusters

- Agent basics and capability model
- Harness Engineering and control layers
- AI coding assistants
- Automation and testing practices

## Practical note-taking pattern

For new material, prefer this sequence:

1. Read and summarize the source.
2. Mark whether the source is conceptual, tactical, or promotional.
3. Link it to an existing topic page.
4. Add one durable insight and one open question.

## What to avoid

- Editing `raw/` during synthesis
- Storing only article summaries without topic pages
- Letting multiple notes say the same thing with different wording
- Mixing tentative claims with stable knowledge without labels

## Next useful expansions

- Add pages for `RAG`, `MCP`, `Context Engineering`, and `multi-agent review`.
- Add a `projects/` subtree if the vault starts tracking concrete implementations.
- Add a `questions/` subtree if you want a persistent backlog of unresolved topics.

## Related pages

- [AI Agent Knowledge Map](./ai-agent-map.md)
- [Agent](../concepts/agent.md)
- [Harness Engineering](../concepts/harness-engineering.md)
- [Claude Code](../tools/claude-code.md)
