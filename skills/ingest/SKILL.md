---
name: ingest
description: Adds a new source to the wiki — code, a Confluence page, a URL, a PDF, or a transcript — or revises existing pages in place when knowledge changes with no new source. Reads the source, discusses takeaways, writes a cited page, updates index.md, revises related entity and concept pages, runs a backlink audit, and appends a log entry. One ingest may touch 10-15 pages. Use when the user drops in a source, says ingest, add, or import, or asks to update, correct, or revise a page.
---

# ingest

The **compile-once** operation. You read a source once, distill it, and fold its
knowledge into the wiki so it never has to be re-derived per query. This skill is
generic: the universal flow lives here; the source-specific mechanics live in
`references/` (read only the one you need).

## Step 0 — orient (always)

Read `.wiki/SCHEMA.md` first and resolve the wiki **ROOT** from its `Path:` field — it
may point outside this project (e.g. `~/wikis/your-project`); **do not assume the wiki is
`.wiki/` just because the anchor lives there.** Read the canonical spec/grammar and
**categories** from `ROOT/SCHEMA.md`; `index.md`, `log.md`, and `pages/` live under
`ROOT/wiki/`, immutable sources under `ROOT/raw/`. Don't restate the grammar from memory.

Then pick a mode:
- **Mode A — a new source is provided** (code, Confluence, URL, PDF, transcript). Default.
- **Mode B — no new source**, you are correcting/revising knowledge already in the wiki.

---

## Mode A — add a new source (the 6-step flow)

1. **Read & distill.** Dispatch by source type (table below). For verbose sources
   (large PDFs, long transcripts, Confluence dumps, whole repos) delegate the raw
   read to a **subagent** so the dump burns the subagent's context and only the
   distilled, cited note returns to yours.
2. **Discuss takeaways** with the user — surface the key claims and where they will land.
3. **Write a cited SOURCE page** in `wiki/pages/<slug>.md` (slug = lowercase-hyphen).
   Frontmatter: `title`, `tags`, `sources`, `updated`. Footnote every
   non-common-knowledge claim per the SCHEMA.md grammar (paragraph granularity,
   never per-sentence). Source pages are the only citable page type.
4. **Update `index.md`** under the right category: `- [[slug]] — one-line summary (ingested YYYY-MM-DD)`.
5. **Revise related ENTITY and CONCEPT pages** across the wiki: extend each
   Description with the new synthesis, add the new page under "Appearances in
   Sources" as `[[source-slug]]`, and link Related Concepts. Expect ~10-15 pages touched.
6. **Append to `log.md`** — `## [YYYY-MM-DD] ingest | Title` with `Pages written:` /
   `Pages updated:` sub-lines — **then run the backlink audit** (below).

### Backlink audit (the most-skipped, highest-value step)
Grep every page for ones that mention the new entities/concepts but do **not** yet
link the new page; add `[[new-slug]]` back-references so links stay bidirectional.

### Source-type dispatch
| Source | Primary reader | Read this reference |
| --- | --- | --- |
| Code on disk / Git repo | native Read · Grep · Glob (git-driven) | `references/ingest-code.md` |
| Confluence / Jira | **official Atlassian Rovo MCP** | `references/ingest-confluence.md` |
| URL · PDF · transcript · local file | native WebFetch · Read | `references/ingest-web-and-files.md` |

---

## Mode B — revise in place (no new source)

1. Locate the affected page(s) via `index.md` + grep.
2. Edit the relevant section **in place** — living-doc rule, never append a dated
   `## update` block.
3. Bump frontmatter `updated`; refresh affected `[[cross-references]]`, the
   `index.md` line, and `overview.md` if the synthesis shifts.
4. Keep every changed claim's footnote resolvable. Quarantine a superseded claim in
   an explicit `> Superseded:` note rather than deleting it silently.
5. Append `## [YYYY-MM-DD] update | page` to `log.md`.

---

## Every ingest path MUST (the universal contract)

- **Preserve the raw source.** Copy the immutable source material into `.wiki/raw/`
  (text/markdown) or `.wiki/assets/` (binaries like PDFs/images) so citations
  resolve forever. Exception: git-tracked code is already immutable at a pinned
  commit — cite `file:line@SHA` instead of duplicating it (see `ingest-code.md`).
- **Capture provenance for citations** in the page frontmatter and footnotes:
  Confluence page ID + space; repo + path + commit SHA; URL + retrieved date;
  PDF filename + page; transcript file + timestamp.
- **Cross-reference** with `[[slug]]` and run the backlink audit.
- **Refresh `index.md`** on every page-changing op.
- **Append to `log.md`** (append-only; ops in {init, ingest, query, update, lint, sync}).

## Hallucination guards
Ground every claim in a verbatim quote from the source; restrict claims to the
provided sources only; if a supporting quote cannot be found, retract or weaken the
claim. When asked something the sources do not cover, say "not in the sources" — do
not invent.
