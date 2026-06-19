---
tags: [adr]
aliases: [ADR-0001]
status: Accepted
date: 2026-06-19
updated: 2026-06-19
---

# ADR-0001. Single shared codebase, per-temple `.env`

- **Status:** Accepted (documenting existing practice)
- **Date:** 2026-06-19
- **Deciders:** ISKCON Noida IT

## Context

DMS runs for multiple ISKCON temples (Greater Noida West, Whitefield, …). Each temple needs its own donors, receipts numbering, branding, contact info, and payment credentials, but the *behavior* is essentially identical everywhere.

The available options were:
1. **Fork the repo per temple** — each temple gets its own codebase.
2. **One codebase, per-deployment configuration** — same code everywhere, differences in config/DB only.

## Decision

Maintain **one Forgejo repo (`IskconNoida/dms`)** deployed to many temples. All temple-specific differences live in:
- the per-deployment **`.env`** (injected by Phase at deploy time), and
- the per-deployment **database** (RDS).

Deployment is `development` → `uat` → `main`, then Jenkins deploys; see [[infrastructure]].

## Consequences

**Easier**
- One place to fix bugs and ship features; every temple benefits at once.
- No per-temple code drift; onboarding a new temple = new `.env` + DB.
- Branding/numbering/contact via env keys (`NATIVE_TEMPLE_CITY`, `RECEIPT_PREFIX`, …).

**Harder / risks**
- A bug ships to **all** temples simultaneously — UAT discipline matters.
- Temple-specific behavior must be expressed as config, not code branches.
- Secrets management is centralized in Phase — its availability is part of the deploy critical path.

> [!warning] Operational implication
> Because one bug affects every temple, the sensitive [[donation-flow#⚠️ Fundraiser attribution|fundraiser-attribution]] code deserves extra review before release.

## Related
[[infrastructure]] · [[architecture-overview]] · [[decisions/README]]
