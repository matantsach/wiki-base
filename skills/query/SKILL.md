---
name: query
description: Answers a question against a wiki built with init and ingest. Reads index.md first, greps and reads only the matching pages, and synthesizes an answer WITH CITATIONS to source pages, raw files, or URLs. Do not answer from general knowledge — always read the wiki pages first, and say so when the answer is not in the wiki. Offers to file valuable answers back as new pages. Use when asking, querying, or searching the wiki.
allowed-tools: Read, Grep, Glob
---

# query — answer from the wiki, with citations

Pure agentic search over markdown — no RAG, no embeddings. You retrieve by reading
`index.md` first, then grep/glob to the few pages that matter, then synthesize a
cited answer. Load only what the question needs; never bulk-load the wiki (context is
finite — avoid context rot).

## The loop

1. **Orient.** Read `.wiki/SCHEMA.md` (resolve the wiki root from its `Path:` field and
   load the citation grammar), then read `wiki/index.md` — the categorized catalog is
   your map.
2. **Find candidates.** `grep`/`glob` the wiki for the question's key terms, entities,
   and slugs. Use the index categories to narrow the search.
3. **Read just-in-time.** Open ONLY the matching pages. Follow `[[wikilinks]]` one hop
   when a page points at the ground truth. Stop when you have enough.
4. **Assess and refine.** If the pages don't answer it, grep again with different terms
   before giving up.
5. **Synthesize with citations.** Answer in the form that fits (prose, comparison
   table, or a Mermaid diagram in a fenced code block). Cite every non-common-knowledge
   claim per the SCHEMA.md grammar — footnotes resolving to **source pages, raw/ files,
   or live URLs** only. Entity, concept, and analysis pages are syntheses and are
   NEVER citation targets.

## Hard rules

- **Do not answer from general knowledge.** Always read the wiki pages first. The wiki
  is the source of truth; your prior knowledge is not.
- **If it isn't in the wiki, say so** — "not in the wiki / not in the sources" — rather
  than invent. Then offer to ingest a source that would cover it.
- **Never fabricate a citation.** A claim with no resolvable footnote does not ship.
- **Codebase wikis:** verify specific values (counts, names, configs, file:line) against
  the live git-tracked source before asserting them — pages can drift from `HEAD`.

## File the answer back (offer)

If the answer is a genuinely new synthesis worth keeping, offer to file it as a new
**analysis** page in `wiki/pages/` (frontmatter title/tags/sources/updated; citations
intact), add its line to `index.md` under the Analyses category, and append to
`wiki/log.md`:

```
## [YYYY-MM-DD] query | <question>
```

This is opt-in and the only write this skill makes — accept the Write prompt at that
point. Filing valuable answers back is how explorations compound instead of being
re-derived next time.
