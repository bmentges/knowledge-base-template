---
name: wiki-ingest
description: Ingest a source document (article, paper, meeting notes, web clipping, transcript, PDF) into an LLM Wiki vault. Use when the user says "ingest this", "add this source", "process this article", "file this", or drops a new file into raw/. Creates a source summary page, updates entity and concept pages, updates wiki/index.md, and appends to wiki/log.md.
---

# wiki-ingest

Process a single raw source into the wiki. One source in → many wiki pages touched.

## Pre-flight

Confirm the current directory is a wiki vault:

- `CLAUDE.md` exists at the root.
- `wiki/` exists with at least `index.md` and `log.md`.
- `raw/` exists.

If any are missing, stop and suggest running `wiki-bootstrap` first.

**Read `CLAUDE.md` before doing anything else.** It defines page types, entity types, and conventions for this specific vault. Everything you write must conform to that schema — not to generic defaults.

## Step 1 — Identify the source

Confirm with the user which source to ingest if there is any ambiguity. Accepted inputs:

- A path under `raw/`.
- A URL → save the fetched content to `raw/` first with a descriptive filename (`YYYY-MM-DD-short-title.md` or original extension for PDFs).
- Attached content → save it to `raw/` first, same naming.

If multiple files in `raw/` have no corresponding page in `wiki/sources/`, list them and ask which to process.

## Step 2 — Read and extract

Read the full source. For image-heavy sources, read the text first, then view referenced images selectively for context. For long PDFs, use page ranges.

Extract:

- **Metadata**: title, author, date, origin (URL or venue), source type.
- **Key claims and facts** — what a reader would need to take away.
- **Entities mentioned** — matching the types defined in `CLAUDE.md`.
- **Relationships** — X depends on Y, A caused B, Z contradicts W, M supersedes N.
- **Open questions or follow-ups**.

## Step 3 — Discuss (optional)

Summarize takeaways to the user in 3–6 bullets. Ask if any angle should be emphasized before filing. Skip this step only if the user has indicated they prefer batch mode.

## Step 4 — Create the source page

Write `wiki/sources/source-YYYY-MM-DD-short-title.md` using the frontmatter defined in `CLAUDE.md`:

- `type: source`
- `confidence: 0.9` (direct reading of a source is high-confidence for *what the source says*, not for whether the source is correct)
- `sources: []` (a source page does not cite itself)
- `tags:` reflect the domain

Body structure:

1. **Metadata block** — author, date, origin, source type.
2. **Summary** — 3–5 tight paragraphs or a bullet list covering the whole source.
3. **Key claims** — one per line; each cite-able from other pages.
4. **Entities introduced** — wikilinks to entity/concept pages.
5. **Open questions** — what this source leaves unresolved.

## Step 5 — Propagate to entity and concept pages

For each entity or concept mentioned in the source:

| Case | Action |
|---|---|
| Page exists and agrees with source | Update: add new facts, add this source to `sources:` frontmatter, bump `updated:`, consider strengthening `confidence`. |
| Page exists and disagrees with source | **Do not overwrite.** Add a `> [!warning]` callout on both the entity page and the new source page noting the contradiction. Flag it in the report at Step 7. |
| Page does not exist and entity is important | Create the page with minimal frontmatter, link to this source, confidence matching single-source support (typically 0.5–0.7). |
| Passing mention, not important | Mention only in the source page. Do not create a stub. |

"Important" means the entity either is central to a claim you'd cite, or is likely to be referenced again. When in doubt, err on the side of not creating stubs — orphans are lint noise.

## Step 6 — Update index and log

Append new pages to `wiki/index.md` under the right category with a one-line summary. Update summaries of modified pages if their essence changed.

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] ingest | Source Title
Created: [[source-YYYY-MM-DD-short-title]], [[new-entity-a]], [[new-concept-b]]
Updated: [[existing-page-c]], [[existing-page-d]]
Flagged: <contradictions, follow-ups, or "none">
```

## Step 7 — Report

Short summary to the user:

- Pages created (count + wikilinks).
- Pages updated (count + wikilinks).
- Contradictions or follow-ups flagged (if any — point to the specific pages).

Stop there. Do not proactively ingest the next source unless the user asks.

## Non-goals

- Do not modify `raw/` files. They are immutable.
- Do not run `wiki-lint` as part of ingest. Lint is a separate operation.
- Do not answer questions about the source beyond the summary — that is `wiki-query`.
- Do not commit to git unless the user asks.
