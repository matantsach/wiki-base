---
name: lint
description: Audits a wiki for health — contradictions, stale or superseded claims, orphan pages, important concepts lacking a page, missing cross-references, broken links, and coverage gaps — and can also fact-check a single page's citations against its sources. Writes a severity-tiered report and logs it. Run after every 5-10 ingests, or when the user says lint, health-check the wiki, or verify a page's citations.
allowed-tools: Read, Grep, Glob, Write, Edit
---

# lint — keep the wiki trustworthy as it grows

A wiki rots silently: links break, claims go stale, pages drift apart. `lint` is the
maintenance pass that catches the rot. It has two modes and is **report-only** — it writes a
NEW report page plus one `log.md` line and never edits existing content or `raw/`. If you spot
a fix, propose it as a diff and apply it only when the user says so.

## Before you start

- Read `.wiki/SCHEMA.md`; resolve the wiki root from its `Path:` field. The conventions and the
  citation grammar you check against live there — follow them, don't assume.
- Pick the mode: **no page named → Mode A** (whole-wiki health). **A specific page named → Mode B**
  (citation audit).
- Work like every other skill: read `index.md` first, then grep/glob for evidence and open pages
  just-in-time. Never bulk-load the wiki.

## Mode A — whole-wiki health check

Gather evidence with grep/glob, classify every finding by severity, then write the report.

### RED — errors (fix before trusting the wiki)
- **Broken `[[slug]]` links** — a wikilink with no matching `wiki/pages/<slug>.md`. Exempt the
  control files (`[[index]]`, `[[log]]`, `[[overview]]`, `[[SCHEMA]]`), which resolve by basename
  outside `pages/` and are valid targets.
- **Missing required frontmatter** — a page lacking `title`, `tags`, `sources`, or `updated`.

### YELLOW — warnings
- **Orphan pages** — 0 inbound `[[links]]` (exclude `index.md` and `overview.md`).
- **Contradictions** — the same entity with conflicting dates, counts, names, or relationships
  across pages.
- **Stale claims** — page `updated` is >90 days old AND the body says "current / latest / recent /
  state-of-the-art" or carries a year literal ≥2 years old.
- **Journal drift** — dated `## [date] update` blocks inside a page that should be integrated in
  place (pages are living documents, not changelogs).

### BLUE — suggestions
- **Missing concept pages** — a `[[slug]]` referenced 3+ times with no page behind it.
- **Coverage gaps** — `overview.md` Open Questions the wiki could now answer via search or a quick
  ingest.
- **Missing cross-references** — pages that mention an entity/concept but don't link its page
  (backlink debt — the most common decay).

### Report
Write `wiki/pages/lint-YYYY-MM-DD.md` with four sections: **Errors / Warnings / Suggestions /
Next Questions**. Each finding states *what*, *where* (page + locator), and a suggested fix.
Append `## [YYYY-MM-DD] lint | health check` to `log.md`.

**Cadence:** run after every 5-10 ingests, or whenever the graph feels tangled.

## Mode B — single-page citation audit

Fact-check one page's claims against the sources it actually cites. (This folds the audit
operation into lint.) Only **source pages** and `raw/` files / URLs are citation targets — entity,
concept, and analysis pages are syntheses and are never cited, so audit the SOURCE pages they draw
from instead.

**Phase A — coverage.** List every non-common-knowledge claim on the page that lacks a footnote.
Common knowledge is exempt.

**Phase B — verification.** Group the page's footnotes by the source file each one resolves to.
Read each source **once** (when there are many, you may dispatch one read-only subagent per source
in parallel so each dump stays out of the main context). For every cited claim, find the supporting
passage and assign a verdict:
- **supported** — locator and quote both check out.
- **partial** — directionally right, but the locator or wording is off.
- **unsupported** — no passage backs the claim.
- **source-missing** — the cited `raw/` file, page, or URL can't be resolved.

Write `wiki/pages/audit-<page-slug>-YYYY-MM-DD.md` listing each claim, its footnote, the verdict,
and the evidence. Append `## [YYYY-MM-DD] lint | citation audit of [[page-slug]]` to `log.md`.

**Any claim whose supporting quote cannot be found must be retracted or weakened — never left
standing.** Flag it in the report and offer the corrected wording as a diff.
