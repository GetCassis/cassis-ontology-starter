---
type: Domain
title: CHANGE ME
description: Global context for <your organization>'s data.
---

Replace this file first. It is the root domain: the context that applies to every question, whatever
the topic. An agent reads it before anything else, so what belongs here is what a new analyst would
need told on day one and never again.

Write, in your own words:

- What the business does, in two or three sentences.
- How the data is organized, and which sub-domain to open for what. Link them: `[sales](sales)`.
- The rules that hold across every question. Currency and how to render it. Which statuses to filter
  and which to keep. Whether timestamps are UTC. Which id counts an entity. Any term that has one
  governed definition and must not be re-derived.

Keep table-level facts out of here — grain, dedup keys, value lists and lineage belong on the table
and column files, which is where an agent looks for them.

For two complete worked examples, a minimal one and a fully authored one, see
[cassis-ontology-examples](https://github.com/GetCassis/cassis-ontology-examples).
