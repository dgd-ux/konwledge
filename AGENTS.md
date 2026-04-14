# LLM Wiki Maintenance Guide

This directory is a local LLM wiki built from project knowledge notes.

## Principles

- Preserve raw source files under `raw/sources/` as immutable inputs.
- Put derived knowledge in `wiki/`.
- Every derived page must include a `来源` section pointing to raw sources.
- Prefer small topical pages over one large summary dump.
- Update `index.md` when adding a page that changes the main navigation.
- Append a dated note to `log.md` for every non-trivial content update.

## Folder Rules

- `raw/`: source material, copied as-is.
- `wiki/source-notes/`: source-oriented notes that restate one raw document.
- `wiki/auth/`: RBAC and permission-system topics.
- `wiki/architecture/`: scheduling, runner, callback, and execution topics.
- `schema/`: naming, taxonomy, and page templates.

## Page Convention

Each derived page should follow this order:

1. Title
2. One-sentence summary
3. Main sections
4. `来源`
5. Optional `待确认`

## Naming

- Use lowercase English file names with hyphens.
- Keep titles human-readable in Chinese.
- Avoid duplicate pages that say the same thing with different wording.

## Update Workflow

1. Copy new raw inputs into `raw/sources/`.
2. Create or update topical pages in `wiki/`.
3. Link them from `index.md`.
4. Record the change in `log.md`.
