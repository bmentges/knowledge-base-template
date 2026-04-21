# {{USE_CASE}} — Knowledge Base

## Purpose

{{PURPOSE}}

## Architecture

Three layers:

- `raw/` — immutable source documents. Never edited, only appended to. Your source of truth.
- `wiki/` — LLM-maintained markdown pages. Cross-referenced, consistent, evolving. The knowledge base.
- This file (`CLAUDE.md`) — the schema. Rules for what goes where, how pages are structured, and how operations work.

## Operations

Driven by three skills:

- **`wiki-ingest`** — when a new file lands in `raw/` or the user says "ingest this" / "add this source".
- **`wiki-query`** — when the user asks a question against the wiki.
- **`wiki-lint`** — periodic health check; or when the user says "lint the wiki".

Every operation must:

1. Update `wiki/index.md` if pages were created, moved, or had their summary materially change.
2. Append an entry to `wiki/log.md` using `## [YYYY-MM-DD] operation | Brief Title`.

## Page types

Every wiki page has one of these types (set in frontmatter `type`):

| Type | Location | Purpose |
|---|---|---|
| `source` | `wiki/sources/` | One-per-raw-document summary. File name: `source-YYYY-MM-DD-short-title.md`. |
| `entity` | `wiki/entities/` | A named thing: {{ENTITY_TYPES}}. |
| `concept` | `wiki/concepts/` | An idea, pattern, framework, or theory. |
| `analysis` | `wiki/analysis/` | Filed query results, comparisons, deep dives. File name: `analysis-YYYY-MM-DD-short-title.md`. |
| `overview` | `wiki/overview.md` | High-level synthesis. One page. |
| `index` | `wiki/index.md` | Catalog of all pages. One page. |
| `log` | `wiki/log.md` | Chronological append-only record. One page. |

## Source types expected

{{SOURCE_TYPES}}

## Frontmatter

Every wiki page opens with:

```yaml
---
title: Page Title
type: entity | concept | source | analysis | overview | index | log
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: 0.0-1.0
sources: [source-YYYY-MM-DD-short-title]
tags: []
---
```

Confidence scale:

| Value | Meaning |
|------:|---------|
| 0.9+  | Verified (multiple independent sources or direct observation) |
| 0.7   | Supported (clear source, no contradiction) |
| 0.5   | Single source |
| 0.3   | Inferred (synthesis, not stated) |
| 0.1   | Speculative (flag for revisit) |

## Formatting rules

- **Wikilinks**: `[[page-name]]` or `[[page-name|Display Text]]`. Never relative markdown links.
- **Callouts**: `> [!note]`, `> [!warning]`, `> [!tip]`.
- **Tags**: YAML frontmatter only — never inline `#tags`.
- **Aliases**: `aliases: [alt-name]` when a page has multiple names.
- **Code blocks**: fenced with language identifiers.
- **File names**: `kebab-case.md`.
- **Mermaid**: use liberally for architecture, flows, sequences. Use `<br>` for line breaks. Never use bullet chars (`•`, `-`, `*`) or numbered lists inside node labels — Obsidian rejects them. Use commas for separation.

## Writing style

- Tables over prose for structured data.
- Thorough in analysis, concise in output.
- Cite source pages via wikilinks: `[[source-YYYY-MM-DD-short-title]]`.
- Cite code with file paths: `github/airflow/dags/some_dag.py`.

## Index and log

- `wiki/index.md` is the catalog. Every page listed with a wikilink, a one-line summary, and its type. Organized by category.
- `wiki/log.md` is chronological and append-only. Parseable with `grep "^## \[" wiki/log.md`.

## Contradictions

When a new claim contradicts an existing claim, do **not** silently overwrite. Note both on the affected pages with a `> [!warning]` callout, keep the older claim with its original source, and flag the contradiction for the user. Resolution is a `wiki-lint` / human decision, not an ingest decision.

## Evolution

This schema is a starting point. Refine it as the vault grows. When you notice a recurring pattern that the schema does not capture (new entity type, new page shape, new convention), update this file and note the change in `wiki/log.md`.
