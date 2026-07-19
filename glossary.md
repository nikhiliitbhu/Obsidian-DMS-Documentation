---
tags: [reference, domain]
aliases: [Glossary, Terms, Definitions]
updated: 2026-06-19
---

# Glossary

Domain, ISKCON, and infrastructure terms used across the DMS code and these docs. Code fields are shown in `monospace`.

> [!note] Confidence
> Definitions tied to code fields are verified against the schema. A few ISKCON acronyms are marked _(unconfirmed)_ — correct them when you know the real expansion rather than guessing.

## Donations & people

| Term | Meaning |
|------|---------|
| **Donation** | The central transaction record (`donation` table) — see [[domain-model]], [[donation-flow]]. |
| **Donor / Devotee** | The giver. `user_id` / `devotee_user_id` on a donation; created with `role_id=2`. A *devotee* is a practitioner/member; donor changes are audited in `devotee_log`. |
| **Fundraiser** | Person/account credited for bringing a donation, tracked by `fundraiser_id`. Drives the **10% incentive**. Defaults to **`111141`** (house account) when none. See [[donation-flow#⚠️ Fundraiser attribution]]. |
| **Collector / Collecting person** | Who physically collected the funds — `collecting_person_name` (free text). Often confused with fundraiser; that confusion is a known bug class. |
| **Facilitator / Counselor** | Additional attribution roles — `facilitator_person_name`, `counselor_person_name`. |
| **Seva** | Devotional *service*; here a category/offering a donation supports. |
| **Nitya seva** | Perpetual/recurring seva (`nitya_seva_table`, `nitya_id`). Often paired with [[payments#E-mandate (recurring debit)\|e-mandate]]. |
| **Project / Head** | The fund/cause a donation supports (`project` table, `head_id`). |
| **Gift** | Item given to a donor (`gift_table`, `gift_id`, `gift_status`). |
| **Sadhna / Sadhana** | Daily spiritual practice (e.g. chanting rounds). The Sadhna mobile app tracks it — see [[api-reference#Sadhna app]]. |
| **ATG receipt** | `atg_receipt` field — a receipt reference on a donation. _(Acronym unconfirmed.)_ |
| **Spiritual role / master / initiated name** | Devotee spiritual identity fields (`spiritual_role` FK, `spiritual_master`, `initiated_name`) — distinct from access [[auth-rbac\|roles]]. |

## Payments

| Term | Meaning |
|------|---------|
| **Razorpay** | Online payment gateway (card/UPI/netbanking). Capture sets donation `status='Closed'`. See [[payments#Razorpay (online)]]. |
| **E-mandate** | Recurring bank auto-debit (ACH/NACH). `emandate_bank_import`, `workflow='Emandate'`. See [[payments#E-mandate (recurring debit)]]. |
| **POS** | Point-of-sale machines for on-site card donations (`pos_machines`). |
| **Workflow / Status** | Two free-text fields describing a donation's pipeline stage — [[donation-flow#Statuses & workflows]]. |

## System & infra

| Term | Meaning |
|------|---------|
| **DMS** | **D**onation **M**anagement **S**ystem — this app (`iskcon-noida/dms`). |
| **Vanguard** | The Laravel starter kit DMS is built on (auth, RBAC, users) — [[auth-rbac]]. |
| **IIC** | The app's public brand/domain (`iic.iskconnoida.org`). _(Acronym expansion unconfirmed.)_ |
| **Forgejo** | Self-hosted git host on the master instance — [[infrastructure#Hosts]]. |
| **Jenkins** | CI/CD that builds and deploys DMS. |
| **Phase** | Secrets manager injecting per-temple `.env` at deploy. |
| **RDS** | AWS managed MySQL (`iic-prd-db`, `iic-master-db`). |
| **myseva** | A **separate** Leantime/Docker app (`myseva.iskconnoida.org`) — *not* this codebase. [[infrastructure#Don't confuse DMS with myseva]]. |
| **Pinbot** | SMS / messaging API (`PINBOT_API_*`). |
| **Pusher** | Real-time broadcasting service. |

## See also
[[domain-model]] · [[donation-flow]] · [[payments]] · [[auth-rbac]] · [[infrastructure]]
