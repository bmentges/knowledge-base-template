---
name: wiki-bootstrap
description: Bootstrap a new LLM Wiki knowledge base in the current directory. Use when the user wants to set up a second brain, research vault, book companion, course notes, business/team knowledge base, or any persistent LLM-maintained wiki. Scaffolds raw/ and wiki/ directories, writes a tailored CLAUDE.md schema, seeds index/log/overview pages, and configures Obsidian defaults. One-shot per vault; subsequent operations use wiki-ingest, wiki-query, and wiki-lint.
---

# wiki-bootstrap

One-shot skill that sets up a new LLM Wiki vault in the current working directory. After bootstrap, the generated `CLAUDE.md` plus the `wiki-ingest` / `wiki-query` / `wiki-lint` skills drive all ongoing operations.

## Pattern reference

The pattern you are applying is the LLM Wiki. Background is bundled with this plugin (read only if needed):

- `origin/llm-wiki.md` — the original pattern (three layers: raw, wiki, schema; three operations: ingest, query, lint).
- `origin/llm-wiki-v2.md` — scaling extensions (confidence, lifecycle, graph, automation).
- `origin/obsidian-conventions.md` — frontmatter, wikilinks, mermaid rules.

You do not need to explain the pattern to the user during bootstrap. Apply it.

## Step 1 — Pre-flight: inspect the directory

Before writing anything, enumerate current state. Run `ls -la` in the working directory and classify:

- **VCS/meta**: `.git/`, `LICENSE`, `README.md`, `.gitignore`, `.editorconfig`
- **Existing Obsidian vault**: `.obsidian/`
- **Existing wiki**: `CLAUDE.md`, `wiki/`, `raw/`
- **Markdown notes**: loose `.md` files that are not the files above
- **Other content**: code, configs, anything else

## Step 2 — Decide placement

Match the directory state to one of these cases and confirm with the user:

| Directory state | Action |
|---|---|
| Empty, or only `.git` / `LICENSE` / `README.md` | Proceed in-place after confirming the use case. |
| Already has `wiki/` + `CLAUDE.md` that looks like a bootstrapped vault | **Stop.** Tell the user this vault is already bootstrapped. Offer: augment (do not re-bootstrap), re-bootstrap anyway (warn: destructive), or pick a different directory. |
| Has markdown notes but no wiki structure | Offer three paths: (a) bootstrap in-place and move existing notes into `raw/`, (b) bootstrap in a subdirectory (e.g. `./wiki-vault/`), (c) pick a different directory. |
| Has unrelated files (code, configs) | Offer: (a) bootstrap in a subdirectory, (b) pick a different directory. Do not mix a wiki into an unrelated project in-place. |

Never overwrite an existing file silently. If a file you plan to create already exists, ask.

## Step 3 — Interview the user

Ask these in one concise message (not one question at a time). Accept free-form answers.

1. **Use case**. Shortlist: personal (goals, journal, self-improvement), research (topic deep-dive), book companion (reading a specific book), course notes, business/team knowledge, competitive analysis, trip planning, other.
2. **Domain specifics**. What kinds of entities will dominate? People, papers, companies, characters, tools, concepts?
3. **Source types**. Primarily articles? PDFs? meeting notes? podcast transcripts? web clippings? transcripts?
4. **Scale expectation**. Handful of sources, dozens, or hundreds+?

Keep it tight. The schema can refine after the first few ingests.

## Step 4 — Present the plan

Before writing any files, list exactly what you will create. Example:

```
I will create:
  raw/                               (empty, with .gitkeep)
  raw/assets/                        (empty, with .gitkeep)
  wiki/index.md
  wiki/log.md
  wiki/overview.md
  wiki/entities/                     (empty, with .gitkeep)
  wiki/concepts/                     (empty, with .gitkeep)
  wiki/sources/                      (empty, with .gitkeep)
  wiki/analysis/                     (empty, with .gitkeep)
  CLAUDE.md                          (tailored to your use case)
  .obsidian/app.json                 (sets attachment folder to raw/assets/)

Proceed?
```

Get explicit confirmation before writing.

## Step 5 — Write the files

Use the templates in `skills/wiki-bootstrap/templates/` as the baseline. Fill in variables from the interview:

- `{{USE_CASE}}` — short label (e.g. `personal`, `research on distributed systems`, `book: The Power Broker`).
- `{{PURPOSE}}` — one-paragraph mission statement you draft from the interview answers.
- `{{ENTITY_TYPES}}` — the dominant entity types the user named (comma-separated).
- `{{SOURCE_TYPES}}` — the source types the user named (comma-separated).
- `{{DATE}}` — today's date in `YYYY-MM-DD`.

Templates to render:

| Template | Destination |
|---|---|
| `templates/claude-md.template.md` | `CLAUDE.md` |
| `templates/index.template.md` | `wiki/index.md` |
| `templates/log.template.md` | `wiki/log.md` |
| `templates/overview.template.md` | `wiki/overview.md` |
| `templates/obsidian-app.template.json` | `.obsidian/app.json` |

Also create empty `.gitkeep` files in `raw/`, `raw/assets/`, `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/analysis/`.

If the user chose to move existing `.md` notes into `raw/` (Step 2), move them now, preserving filenames.

## Step 6 — Report and hand off

After writing, report concisely:

- Files created (list).
- Entry appended to `wiki/log.md`: `## [{{DATE}}] bootstrap | Wiki initialized for {{USE_CASE}}`.
- Next steps for the user:
  1. Open the directory as an Obsidian vault (Open folder as vault).
  2. Drop a first source into `raw/`.
  3. Ask to ingest it — the `wiki-ingest` skill will take over.
  4. Ask questions — `wiki-query` answers from the vault.
  5. Periodically ask to lint the wiki — `wiki-lint` keeps it healthy.

Stop there. Do not proactively continue into an ingest in the same turn.

## Non-goals

- Do not install Obsidian plugins programmatically.
- Do not fetch sources from the web during bootstrap.
- Do not pre-populate entity or concept pages. Let them emerge from actual sources.
- Do not commit to git. Let the user decide when.
