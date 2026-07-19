---
tags: [adr, moc]
aliases: [ADRs, Decisions, Architecture Decision Records]
updated: 2026-06-19
---

# Architecture Decision Records (ADRs)

An ADR captures **one significant decision**: the context, the choice, and the consequences. They're short, append-only, and never deleted — if a decision changes, write a new ADR that supersedes the old one.

> [!tip] Why ADRs
> Prose docs drift. A dated log of *why* a choice was made survives team turnover and stops the same debate happening twice.

## How to add one

1. Copy [[template]] to `NNNN-short-title.md` (next number, zero-padded).
2. Fill in Context / Decision / Consequences.
3. Set `status:` `Proposed` → `Accepted` → (later) `Superseded by NNNN`.
4. Link it from related docs and add it to the index below.

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [[decisions/0001-shared-codebase-per-temple-env\|0001]] | Single shared codebase, per-temple `.env` | Accepted |

## See also
[[architecture-overview]] · [[infrastructure]]
