---
tags: [overview, synthesis]
updated: 2026-06-26
---

# Overview

A single evolving synthesis of everything this wiki knows. Updated whenever an ingest or a
revision shifts the big picture.

## Current Understanding

This demo wiki holds one source: Andrej Karpathy's "LLM Wiki" idea. The core move is to
**compile knowledge once into interlinked markdown and then keep it current**, rather than
re-deriving it from raw chunks on every query (RAG). The result is a persistent, compounding
artifact — the cross-references are already in place, contradictions are already flagged, and
the synthesis already reflects everything read so far. It is "just a git repo of markdown
files," so version history, branching, and collaboration come for free. See [[example-source]].

## Open Questions

- How does index-first agentic search scale past ~100 sources / hundreds of pages?
- When is an optional markdown search engine (e.g. qmd) worth adding on top of `index.md`?

## Key Entities / Concepts

- [[example-source]] — the seed source for this demo.
