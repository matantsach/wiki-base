# Ingest: Confluence (and Jira) via the official Atlassian MCP server

Pull a Confluence page (or Jira issue) into the wiki using the **official Atlassian
Rovo Remote MCP server** — the vendor's own, supported connector. No scraping, no
bespoke REST client, no stored credentials in the repo.

## The server (cite this)

- Endpoint: `https://mcp.atlassian.com/v1/mcp/authv2`
- Auth: **OAuth 2.1** (browser consent). **Atlassian Cloud only** (not Server/Data Center).
- Docs:
  - Getting started: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/
  - Supported tools: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/supported-tools/
- MCP is the vendor-neutral standard for exactly this:
  https://www.anthropic.com/news/model-context-protocol

## Setup (opt-in — the user runs this, you don't ship it)

Add the server to Claude Code yourself. Do **not** commit an active `.mcp.json`, or
it would trigger an OAuth flow for everyone on load:
```
claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/mcp/authv2
```
Reference: https://code.claude.com/docs/en/mcp

## Tool flow (search → locate → fetch)

1. `searchConfluenceUsingCql` — find candidate pages by CQL (title/space/text/label).
2. `getPagesInConfluenceSpace` — list pages within the target space to confirm the right one.
3. `getConfluencePage` — fetch the full page content (the body you will distill).

(Jira mirrors this with the issue/search tools; same flow.)

## Distill in a subagent

Confluence bodies are verbose (macros, tables, comments). Delegate the
`getConfluencePage` fetch + first-pass distillation to a **subagent** so the dump
stays out of your main context; only the cited summary returns.

## Preserve the raw source

Save a markdown snapshot of the fetched page into `.wiki/raw/` (e.g.
`raw/confluence-<page-id>.md`) so citations resolve even if the live page changes or
access is later lost. This is the immutable layer — never edit it after capture.

## Provenance (put in page frontmatter + footnotes)

Capture: **Confluence page ID**, **space key**, **page title**, **page URL**, and the
**retrieved date**. Example frontmatter:
```yaml
sources:
  - confluence: { space: ENG, page_id: "131072", url: "https://acme.atlassian.net/wiki/spaces/ENG/pages/131072", retrieved: 2026-06-26 }
```

## Citation forms (per SCHEMA.md grammar)

Cite the immutable snapshot or the live page, always with a locator:
```
[^1]: raw/confluence-131072.md §"Rollout plan" — "we cut over region by region"
[^2]: <https://acme.atlassian.net/wiki/spaces/ENG/pages/131072> (2026-06-26)
```

Then run `SKILL.md` Mode A steps 3-6: write the cited SOURCE page → update `index.md`
→ revise related entity/concept pages → append `log.md` + backlink audit.
