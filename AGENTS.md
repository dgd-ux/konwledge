# Knowledge Base Schema

This repository follows a three-layer knowledge workflow:

1. `raw/`
   - Immutable source material.
   - Contains clipped articles, notes, transcripts, PDFs converted to markdown, and other primary captures.
   - Do not edit files in `raw/` unless the task is explicitly about source cleanup.
2. `wiki/`
   - LLM-maintained knowledge layer.
   - Stores distilled source pages, topic pages, synthesis notes, and practice guides.
   - Each page should be updated incrementally instead of rewritten from scratch unless structure is broken.
3. Root control files
   - `index.md`: current map of the knowledge base.
   - `log.md`: append-only activity log for ingestion, restructuring, and major edits.

## Core Rules

- Treat `raw/` as the source of truth.
- Write all derived understanding into `wiki/`.
- Prefer one source page per raw file under `wiki/sources/`.
- Prefer one durable concept/tool/practice page per topic under `wiki/`.
- Preserve uncertainty. If a source is weak, promotional, or ambiguous, say so explicitly.
- Do not duplicate long source excerpts. Summarize and link instead.
- Update `index.md` whenever pages are added, removed, or reorganized.
- Append a short entry to `log.md` for each meaningful ingestion or refactor.

## Page Types

### `wiki/sources/*.md`

Use for source-grounded notes.

Suggested structure:

- Title
- Source metadata
- Why it matters
- Key points
- Caveats
- Related pages

### `wiki/concepts/*.md`

Use for stable concepts such as Agent, RAG, Harness Engineering, MCP.

Suggested structure:

- Definition
- Core components
- What it is not
- Practical implications
- Related tools or methods
- Sources

### `wiki/tools/*.md`

Use for specific products or frameworks.

Suggested structure:

- What it is
- Core capabilities
- Setup or usage patterns
- Limits or risks
- When to use
- Sources

### `wiki/overview/*.md`

Use for maps, playbooks, synthesis notes, and vault-level guidance.

Suggested structure:

- Goal
- Scope
- Recommended workflow
- Reading paths
- Links out to concept/tool/source pages

## Operating Modes

### Ingest

When new material appears in `raw/`:

1. Read the source.
2. Create or update a page in `wiki/sources/`.
3. Update affected topic pages in `wiki/concepts/`, `wiki/tools/`, or `wiki/overview/`.
4. Update `index.md`.
5. Append an entry to `log.md`.

### Query

When answering questions from this repository:

1. Prefer `wiki/` over `raw/`.
2. If the answer requires fresh synthesis, write the durable result back into `wiki/`.
3. If the answer depends on a weak source, mark that limitation.

### Lint

Before finishing a knowledge-base edit:

- Check that every new page has at least one inbound or outbound link.
- Check that topic pages cite their source pages.
- Check that `index.md` and `log.md` were updated.
- Check that `raw/` was not modified unintentionally.

## Naming Conventions

- Use lowercase kebab-case file names.
- Keep titles human-readable inside the markdown body.
- Prefer stable topical files over date-stamped duplicates.

## Current Vault Focus

This vault currently emphasizes:

- Agent basics
- Harness Engineering
- AI coding tools
- Automated testing practice
- Personal knowledge base workflows for AI-native learning
