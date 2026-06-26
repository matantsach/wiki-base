---
title: "Karpathy — the LLM Wiki idea"
tags: [source]
sources: ["https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f"]
updated: 2026-06-26
type: source
---

# Karpathy — the LLM Wiki idea

A short worked-example **source page**. Source pages are the only citable page type: each one
summarizes a single ingested document and grounds its claims in footnotes.

The "LLM Wiki" pattern has an LLM incrementally build and maintain a persistent wiki of
interlinked markdown files that sits between the user and the raw sources. Knowledge is
compiled once and then kept current — not re-derived on every query — so the wiki becomes "a
persistent, compounding artifact."[^1] The whole thing is "just a git repo of markdown
files," which gives you version history, branching, and collaboration for free.[^2]

Citations sit at paragraph granularity (not one per sentence), and common-knowledge statements
need no footnote. For the running synthesis across all sources, see the wiki
[[overview|running overview]].

[^1]: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f> (retrieved 2026-06-26) — "the wiki is a persistent, compounding artifact"
[^2]: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f> (retrieved 2026-06-26) — "just a git repo of markdown files"
