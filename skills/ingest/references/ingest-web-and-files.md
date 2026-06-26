# Ingest: web pages, PDFs, transcripts, and local files

The everyday path. Native readers cover URLs, PDFs, transcripts, and local docs — no
connector, no infra. An official MCP server is an opt-in fallback only.

## Readers by type

- **Web URL** → native **WebFetch** (fetches HTML, converts to markdown). For long
  pages use `start_index` to page through; fetch in chunks rather than truncating.
- **PDF** → native **Read**. Limits: ~32 MB, ~100 pages within a 200k context
  (~600 at larger limits), **no encryption**. Split oversized/encrypted PDFs first.
  Reference: https://platform.claude.com/docs/en/build-with-claude/pdf-support
- **Transcript** (meeting/video, `.vtt`/`.srt`/`.txt`) → native **Read**. Keep the
  `[HH:MM:SS]` timestamps — they are the citation locators.
- **Local docs** (`.md` `.txt` `.csv` `.docx`) → native **Read**.
- **Opt-in fallback:** if a file lives in a sandboxed/remote store the native tools
  can't reach, use the official **MCP filesystem** server rather than a custom
  reader. Add it yourself; don't ship an active config:
  https://code.claude.com/docs/en/mcp

## Distill in a subagent

For big PDFs and long transcripts, delegate the read to a **subagent** so the full
text burns the subagent's context and only the distilled, cited note returns to yours.

## Preserve the raw source (every type)

- **URL** → save the fetched markdown into `.wiki/raw/` (e.g. `raw/<slug>.md`); record
  the canonical URL + retrieved date. Web pages drift, so snapshot what you cited.
- **PDF** → store the file in `.wiki/assets/`; optionally a text excerpt in `.wiki/raw/`.
- **Transcript / local doc** → copy the file into `.wiki/raw/` unchanged.

The `raw/` and `assets/` layers are immutable — read them, never rewrite them.

## Provenance (frontmatter + footnotes)

| Type | Capture |
| --- | --- |
| URL | canonical URL + retrieved date (and section anchor if present) |
| PDF | filename + page number(s) |
| Transcript | filename + `[HH:MM:SS]` timestamp |
| Local doc | filename + section/line |

## Citation forms (per SCHEMA.md grammar — locator mandatory)

```
[^1]: <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents> (2026-06-26)
[^2]: assets/q2-review.pdf p.7 — "retention rose to 41%"
[^3]: raw/standup-2026-06-24.vtt [00:12:31] — "we're cutting the legacy path"
```

Use a **Quote** footnote (verbatim) for specific facts; a `[synthesis]` footnote when
the claim spans a range. Cite at paragraph/claim granularity, never per sentence.

Then run `SKILL.md` Mode A steps 3-6: write the cited SOURCE page → update `index.md`
→ revise related entity/concept pages → append `log.md` + backlink audit.
