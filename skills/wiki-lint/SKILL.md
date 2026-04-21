---
name: wiki-lint
description: Health-check an LLM Wiki vault. Use when the user says "lint the wiki", "health check the wiki", "clean up the wiki", or on a periodic schedule to keep the wiki healthy as it grows. Finds orphan pages, broken wikilinks, stale claims, missing cross-references, contradictions, frontmatter drift, and index staleness. Auto-fixes safe issues, flags the rest for the user.
---

# wiki-lint

Keep the wiki healthy as it grows. Auto-fix what is safe; surface the rest for the user to decide.

## Pre-flight

Confirm the current directory is a wiki vault:

- `CLAUDE.md`, `wiki/index.md`, `wiki/log.md` all exist.

If any are missing, stop and suggest running `wiki-bootstrap` first.

**Read `CLAUDE.md` before linting.** The schema defines what "valid" means for this vault — page types, frontmatter fields, allowed entity categories. Do not apply generic defaults over vault-specific rules.

## Auto-fix vs flag

Distinguish two classes of issue:

- **Auto-fix**: mechanical, reversible, no judgment needed (e.g. add missing `created:` field, normalize frontmatter order, add an obvious index entry). Apply directly.
- **Flag**: requires judgment, affects meaning, or could lose information (e.g. delete orphan, resolve contradiction, rename page). Surface to the user with a recommendation; do not apply.

When uncertain, flag. It is better to over-flag than to silently change meaning.

## Checks to run

### 1. Frontmatter drift

For every page in `wiki/`, verify frontmatter matches `CLAUDE.md`:

- Required fields present: `title`, `type`, `created`, `updated`, `confidence`.
- `type` value in the allowed set for this vault.
- `updated` not older than the newest source in `sources:` (otherwise, stale).

**Auto-fix**: add missing `created`/`updated` (use file mtime), missing `confidence` (use 0.5 as conservative default), normalize field order.
**Flag**: unknown `type`, structural mismatches, stale `updated`.

### 2. Broken wikilinks

For every `[[target]]` anywhere in `wiki/`, verify the target exists as a page (or as an alias in some page's `aliases:`).

**Auto-fix**: none by default — renames are judgment calls.
**Flag**: missing targets. If the target is a close match to an existing page name (Levenshtein ≤ 2, or one is a substring of the other), recommend the rename; do not apply.

### 3. Orphan pages

Pages in `wiki/` with zero inbound wikilinks from anywhere under `wiki/` (excluding `index.md` and `log.md`, which don't count as meaningful incoming).

**Flag**: for each orphan, recommend either (a) a specific existing page that should link to it, based on topic overlap, or (b) merging into a parent, or (c) deleting.

### 4. Missing cross-references

Scan prose for entity/concept names (case-sensitive and case-insensitive exact matches against existing page titles and aliases) that are not wrapped in `[[…]]`.

**Auto-fix**: convert unambiguous exact matches to wikilinks.
**Flag**: ambiguous cases (same name could match multiple pages).

### 5. Contradictions

Compare claims across pages that share entities. Look for:

- Numeric mismatches on the same attribute.
- Directly opposing statements about the same entity.
- A page claiming X and a more recent source page claiming not-X, without a supersession callout.

**Flag only — never auto-resolve.** For each, present both claims, their sources, their dates, and a recommendation based on recency and source count. The user decides.

### 6. Index freshness

- Every page under `wiki/` (except `index.md`, `log.md`) should appear in `wiki/index.md`.
- Every entry in `wiki/index.md` should point to a real page.

**Auto-fix**: add missing entries under the correct category, remove dead entries, update one-line summaries that no longer match the page's current content.

### 7. Overview staleness

If `wiki/overview.md`'s `updated:` date is older than the most recent ingest in `wiki/log.md`, the overview likely needs a rewrite.

**Flag**: recommend what recent material warrants inclusion. Do not auto-rewrite.

## Output — present the report

Group findings by check. Include counts and lists. Keep it scannable.

```
Lint report — YYYY-MM-DD

1. Frontmatter drift          3 issues (2 auto-fixed, 1 flagged)
2. Broken wikilinks           4 issues (0 auto-fixed, 4 flagged)
3. Orphan pages               2 flagged
4. Missing cross-references   7 issues (5 auto-fixed, 2 flagged)
5. Contradictions             1 flagged
6. Index freshness            3 auto-fixed
7. Overview staleness         1 flagged

Total: 21 issues, 10 auto-fixed, 11 flagged for review.
```

Then for each flagged item: page path, issue, recommended action. Keep recommendations short and specific.

## Log

Append to `wiki/log.md`:

```
## [YYYY-MM-DD] lint | N issues found, M auto-fixed
Flagged: <short summary with wikilinks to affected pages>
```

## Non-goals

- Do not delete pages without explicit user confirmation.
- Do not resolve contradictions unilaterally.
- Do not re-ingest raw sources. If a source summary page looks wrong, flag it and suggest running `wiki-ingest` again on that source.
- Do not modify `raw/`. It is immutable.
- Do not commit to git unless the user asks.
