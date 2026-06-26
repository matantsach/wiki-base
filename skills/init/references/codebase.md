# codebase.md — authoring a wiki *for a codebase*

Read this when `init` set **Domain: codebase** (or when ingesting code). It covers how to
*author* codebase pages; the *ingest mechanics* (readers, git raw-layer, sync) live in
`skills/ingest/references/ingest-code.md` and `codebase-sync.md`. The generic conventions and
citation grammar still come from `SCHEMA.md` — this only adds the code-specific layer.

## Categories & slug prefixes

A codebase wiki replaces the research categories with four, each a slug prefix so the flat
`pages/` dir stays scannable:

| Category  | Prefix  | A page answers… |
| --------- | ------- | --------------- |
| Modules   | `mod-`  | What is this component, what does it own, where does it live? |
| APIs      | `api-`  | What's the contract — inputs, outputs, errors, callers? |
| Decisions | `dec-`  | Why is it this way? What was traded off? |
| Flows     | `flow-` | How does a request / job move across modules end to end? |

`index.md` groups pages under `Modules | APIs | Decisions | Flows` (set by `init`).

## Reader-question-first index

Don't catalog files — catalog the **questions a new engineer asks**. Each `index.md` line is a
question answered by exactly one page:

```
## Flows
- [[flow-auth-login]] — How does a login request become a session? _(updated 2026-06-26)_
## Decisions
- [[dec-event-sourcing]] — Why event sourcing instead of CRUD? _(updated 2026-06-26)_
```

This keeps the wiki **rationale and relationships**, not a second copy of the code. (Credit:
yysun's git-wiki reader-question index.)

## Depth over breadth

**Five deep pages beat fifty shallow ones.** Write a page only when it answers a real question;
go only as deep as a question demands. A `mod-` page that just restates the file tree is noise —
delete it. Prefer one `flow-` page that ties five modules together over five stubs.

## Two flavors of decision page

- **Sourced** — backed by a formal ADR / RFC / design doc. Cite it like any source.
- **Reconstructed** — inferred from the code itself (no written record). Often the *most*
  valuable pages. Mark them clearly and cite `path/to/file.ts:line@<short-SHA>` so a reader can
  verify the inference against the live code.

## README vs wiki — link, don't duplicate

Keep the boundary clean: the repo's **README/docs are operational** (how to set up, build, run,
test). The **wiki is rationale, relationships, and tradeoffs**. When the two would overlap, the
wiki *links* to the README section rather than copying it — copies rot independently.

## Codebase-only frontmatter (enables stale-detection)

Code pages add two fields to the standard `SCHEMA.md` frontmatter so `sync` can tell which pages
a commit touched (credit: yysun's git-wiki):

```yaml
source_paths: [src/api/routes.ts, src/api/auth.ts]   # code files this page documents
stale: false                                          # set true when a source file is deleted
```

`index.md` frontmatter carries the sync checkpoint:

```yaml
last_commit: <full-SHA>   # advanced only after a whole changeset is processed
```

The raw layer for code is **git itself** — don't copy source files into `raw/`; cite a pinned
`file:line@SHA` instead (see `ingest-code.md`). Keeping a code wiki current as commits land is
the `sync` flow in `codebase-sync.md`.

## Sources

- kfchou/wiki-skills — `codebase.md` (MIT): the `mod-`/`api-`/`dec-`/`flow-` taxonomy,
  "five deep pages beat fifty shallow ones," the Sourced-vs-Reconstructed split, and the
  README-vs-wiki boundary. <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-init/codebase.md>
- yysun — `git-wiki/SKILL.md` (MIT): the reader-question-first index and the
  `source_paths[]` / `stale` codebase frontmatter.
  <https://raw.githubusercontent.com/yysun/awesome-agent-world/main/skills/git-wiki/SKILL.md>
