---
tags: [meta]
date: 2026-07-28
---

# Knowledge Base

A personal reference for JavaScript, programming concepts, tools, and learning notes.

## Structure

- `javascript/patterns/` — decision-driven notes: when to use X vs Y, and why (`reduce-vs-for-loop.md`)
- `javascript/mental-models/` — how things actually work under the hood (`event-loop.md`, `closures.md`)
- `javascript/ecosystem/` — tooling tradeoffs (npm vs pnpm, bundler choices)
- `javascript/snippets/` — reusable code, minimal prose
- `general/` — cross-language judgment calls (algorithms, git, design patterns)
- `tools/` — editor configs, CLI tricks, debugging notes
- `learning/` — course notes, book notes, TIL-style entries, and `to-revisit.md` (concepts not yet fully internalized)

## Hub notes (start here to refresh a cluster)

- [shared-mutable-references](javascript/mental-models/shared-mutable-references.md) — connects [reference-equality](javascript/mental-models/reference-equality.md) and [null-prototype-objects](javascript/mental-models/null-prototype-objects.md) (prototype pollution) under one root cause: objects are shared by reference, and mutating a reference mutates it everywhere that reference is held.

## Conventions

- Name files after the question/decision they resolve (`reduce-vs-for-loop.md`), not the API feature (`array.md`). Mental-model notes name after the concept instead (`closures.md`), since there's no "vs" to phrase.
- Lead with the tradeoff or gotcha, then the example — syntax is a footnote, not the point.
- Frontmatter on every file: `tags:`, `date:`, `type:` (`pattern` | `mental-model` | `gotcha` | `snippet`)
- Flat within each folder — avoid deep nesting
