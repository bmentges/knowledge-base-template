# Obsidian & Wiki Conventions

Reference conventions for pages inside a wiki vault bootstrapped by this plugin. The `wiki-bootstrap` skill inlines a tailored version of these rules into each vault's `CLAUDE.md`. This file is the canonical origin.

## Obsidian markdown

- **Wikilinks**: `[[page-name]]` or `[[page-name|Display Text]]`. Never relative markdown links.
- **Callouts**: `> [!note]`, `> [!warning]`, `> [!tip]`.
- **Tags**: YAML frontmatter `tags: [tag1, tag2]`. Not inline `#tags`.
- **Aliases**: `aliases: [alt-name]` for pages with multiple names.
- **Code blocks**: fenced with language identifiers.

## Mermaid diagrams

Use liberally for architecture, data flows, sequences, timelines.

- Use `<br>` for line breaks inside node labels.
- Never use bullet characters (`•`, `-`, `*`) or numbered lists (`1.`, `2.`) inside node labels — Obsidian rejects these as "Unsupported markdown: list".
- Use comma-separated text instead: `"_id→id, TO_JSON_STRING,<br>soft-delete filter"`.

## Page frontmatter

Every wiki page must open with:

```yaml
---
title: Page Title
type: entity | concept | source | analysis | overview | index | log
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: 0.0-1.0
sources: []
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

## Naming

- Files: `kebab-case.md`.
- Source pages: `source-YYYY-MM-DD-short-title.md`.
- Analysis pages: `analysis-YYYY-MM-DD-short-title.md`.
- Log entries: `## [YYYY-MM-DD] operation | Brief Title`.

## Writing style

- Tables over prose for structured data.
- Thorough in analysis, concise in output.
- Cite source pages via wikilinks when making claims.
- Cite file paths when referencing code: `github/airflow/dags/some_dag.py`.
- Always update `wiki/index.md` and `wiki/log.md` after creating, moving, or materially changing pages.
