---
tags: [moc, home]
aliases: [Home, Documentation Home, DMS Docs]
updated: 2026-06-19
---

# 🏛️ DMS Documentation

Engineering documentation for the **Donation Management System** (`iskcon-noida/dms`) — a Laravel 9 / PHP 8 application, built on the [Vanguard](https://vanguardapp.io) starter, that manages donations, donors, projects/seva, receipts, and reporting for ISKCON temples. **One codebase serves many temples**, differing only by `.env` and database.

> [!info] This is an Obsidian vault
> Every file is plain Markdown. Open this folder as a vault in [Obsidian](https://obsidian.md) for backlinks + graph view, or read it on Forgejo/GitHub — the `[[wiki-links]]` and Mermaid diagrams render in both. Press `Ctrl+G` for the graph.

## 🗺️ Map of content

### Understand the system
- [[architecture-overview]] — components, request flow, code layout
- [[CRM and Other Tools]] — full module inventory (it's much more than donations)
- [[domain-model]] — entities + ER diagrams (built from real migrations)
- [[glossary]] — ISKCON + DMS terms (seva, fundraiser, e-mandate…)

### Core flows
- [[donation-flow]] — how a donation moves from entry → payment → receipt
- [[payments]] — Razorpay online capture + e-mandate recurring debit
- [[auth-rbac]] — login, roles, permissions (Vanguard)

### Build & operate
- [[development]] — local setup: install, `.env`, migrate, seed, run
- [[infrastructure]] — AWS hosts, RDS, Forgejo/Jenkins/Phase pipeline
- [[api-reference]] — REST endpoints (web app API + Sadhna app)

### Decisions
- [[decisions/README|Architecture Decision Records]] — why things are the way they are
  - [[decisions/0001-shared-codebase-per-temple-env|ADR-0001: Shared codebase, per-temple .env]]

## 🧭 Where do I start?

| I want to… | Go to |
|------------|-------|
| Run it locally | [[development]] |
| Understand the data | [[domain-model]] |
| Trace a donation | [[donation-flow]] |
| Touch payment code | [[payments]] ⚠️ |
| Understand permissions | [[auth-rbac]] |
| Call the API | [[api-reference]] |
| Know a temple term | [[glossary]] |

## ✍️ Conventions

> [!tip] Keep the vault healthy
> - **One topic per file.** Link generously with `[[file-name]]`.
> - **Diagrams** use Mermaid (renders in Obsidian + Forgejo).
> - **Significant choices** become an ADR in [[decisions/README|decisions/]] — don't bury them in prose.
> - **Tags** group notes: `#architecture` `#domain` `#payments` `#auth` `#api` `#ops` `#reference` `#adr`.
> - **Callouts** (`> [!warning]`) flag risks — especially around [[donation-flow#⚠️ Fundraiser attribution|fundraiser attribution]].

> [!warning] Status: not yet committed
> These docs live in the working tree but may be untracked in git. Commit them so the team gets them:
> ```bash
> git add docs/ && git commit -m "Add architecture documentation"
> ```
