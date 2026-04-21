---
name: wiki-query
description: Answer a question by searching and synthesizing across an LLM Wiki vault. Use when the user asks "what does the wiki say about…", "query the wiki", "ask the wiki", or any substantive question that should be answered from the vault's accumulated knowledge rather than from general training data. Substantive answers can be filed back as analysis pages so explorations compound.
---

# wiki-query

Synthesize an answer from the wiki's contents. Good answers should be filable back into the vault so explorations compound over time.

## Pre-flight

Confirm the current directory is a wiki vault:

- `CLAUDE.md`, `wiki/index.md`, `wiki/log.md` all exist.

If any are missing, stop and suggest running `wiki-bootstrap` first.

**Read `CLAUDE.md` before answering.** It defines the page types, confidence scale, and conventions this vault uses. Citation format and filing rules come from there.

## Step 1 — Locate relevant pages

Start with `wiki/index.md`. Use its categories and one-line summaries to identify candidate pages. This is fast and works well up to a few hundred pages.

If the index is not enough (e.g. the question is about a concept not yet catalogued, or the vault has grown past ~200 pages), also grep across `wiki/` for key terms from the question.

Pull 3–8 candidate pages initially. Expand only if coverage is clearly thin. Do not pre-fetch the whole vault.

## Step 2 — Synthesize

Answer using **only** content present in the wiki and the raw sources it references. Cite everything:

- Cite source pages via wikilinks: `[[source-YYYY-MM-DD-short-title]]`.
- Cite entity/concept pages: `[[page-name]]`.

Confidence rules:

- Your answer's confidence is capped by the confidence of the underlying pages.
- If the wiki lacks coverage on part of the question, say so explicitly. Never fabricate.
- If underlying pages contradict each other, surface the contradiction rather than picking a side silently.

Gap handling — if the wiki can't answer the question, offer one or both of:

- Specific raw sources to ingest to close the gap.
- A web search (only if the user enables it).

## Step 3 — Choose an output format

Match format to the question. Obsidian renders mermaid, tables, and callouts natively — use them when they clarify.

| Question shape | Recommended format |
|---|---|
| "What is X?" / "Tell me about Y" | Prose summary with wikilinks |
| "Compare X and Y" | Table |
| "What changed when?" | Timeline or dated list |
| "What depends on X?" / "What does X touch?" | List, or mermaid dependency graph |
| "Walk me through …" | Ordered steps |
| "Summarize everything about …" | Short narrative + bullet citations |

## Step 4 — Offer to file substantive answers

If the answer synthesizes multiple pages or contains reasoning that would be costly to re-derive, offer to file it:

> File this as an analysis page? (y/n)

If **yes**:

- Write `wiki/analysis/analysis-YYYY-MM-DD-short-title.md` with full frontmatter (`type: analysis`, confidence matching the weakest underlying source).
- Include the original question, the answer, and a "Sources" section listing every wikilink used.
- Update `wiki/index.md` under the Analysis category.
- Append to `wiki/log.md`:

```
## [YYYY-MM-DD] query | Brief Title
Filed: [[analysis-YYYY-MM-DD-short-title]]
From: [[page-a]], [[page-b]], [[page-c]]
```

If **no** or the answer is a simple lookup: append a lighter log entry:

```
## [YYYY-MM-DD] query | Brief Title
Answered from: [[page-a]], [[page-b]]
```

## Non-goals

- Do not invent entities not present in the wiki.
- Do not re-ingest raw sources mid-query — that is `wiki-ingest`'s job. If you notice the wiki is missing a source the user has, tell them.
- Do not rewrite existing pages during a query. If something looks wrong, propose an edit or suggest `wiki-lint`; do not silently edit.
- Do not answer from general training data when the wiki has opinions. The wiki is the source of truth for this domain.
