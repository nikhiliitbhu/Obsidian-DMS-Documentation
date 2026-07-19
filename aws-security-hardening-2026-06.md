# AWS Security & Infrastructure Hardening — ISKCON Noida

**Prepared for:** Project Manager
**Prepared by:** Nikhil (Tech)
**Date:** 22 June 2026
**AWS Account:** ISKCON NOIDA — `891377382138`
**Region:** Asia Pacific (Mumbai) — `ap-south-1`

---

## At a Glance — One-Page Summary

**Context:** Following the June 2026 database-corruption incident ("DevZilla"), we hardened the AWS account's **backups, audit logging, threat detection, monitoring, and access controls.**

**What we put in place (all live unless noted):**

| Area | What we did | Status |
|---|---|---|
| 💾 **Backups** | Production DB: **30 daily + 24 monthly** (2-year) snapshots; Master DB: **30 daily**; both with accidental-deletion protection + 1-day point-in-time recovery | ✅ |
| 📜 **Audit logging** | **CloudTrail** — permanent, tamper-evident record of all account activity (*there was none before*) | ✅ |
| 🛡️ **Threat detection** | **GuardDuty** — automated detection of suspicious activity / intrusions | ✅ (free 30-day trial) |
| 📊 **Monitoring & alerts** | **4 CloudWatch alarms** (DB storage + CPU) emailing the team via SNS | ✅ (1 of 2 emails confirmed) |
| 🔑 **Access review** | Root secured (MFA, no keys); admin MFA on; DevZilla suspect account gone | ✅ |

**Cost impact:** ≈ **$6–16 / month** (≈ ₹500–1,350) — almost entirely GuardDuty; everything else is free or a few cents.

**Biggest improvements:** 7-day backup window → **2-year archive**; **no audit trail → permanent tamper-evident logging**; **no monitoring → automated email alerts**.

**Still pending (team):** MFA for 4 staff logins · rotate old access keys · confirm 2nd alert email (`jgd@`) · remove 1 dormant user.

---

## 1. Executive Summary

Following the June 2026 database-corruption incident ("DevZilla"), we reviewed the AWS account's backup, audit, monitoring, and access-control posture and closed the major gaps. This document records **what was put in place, what each feature does, its effect on the monthly bill, current status, and the items still outstanding** (being handled by the team).

**Headline outcomes:**

- **Backups:** Production and master databases now have automated, long-retention backups + accidental-deletion protection. Previously there was only a 7-day window.
- **Audit logging:** A durable, tamper-evident record of all account activity (CloudTrail) now exists. Previously there was **no trail at all** — only a rolling 90-day history.
- **Threat detection:** Automated threat detection (GuardDuty) is now active. Previously none.
- **Monitoring & alerts:** Database health alarms now email the team automatically. Previously there were **zero** alarms and no notification channel.
- **Estimated added cost:** **≈ $6–16 / month** (≈ ₹500–1,350/month), almost entirely GuardDuty; everything else is free or a few cents.

---

## 2. Features Implemented

### 2.1 Database Backups — AWS Backup

**What it is:** A managed backup service that takes scheduled snapshots of our RDS databases and keeps them for a defined period, independent of the database itself.

**Why we needed it:** Native RDS automated backups max out at 35 days and cannot do long-term/monthly archives. AWS Backup gives us a proper retention scheme.

**What we configured:**

| Database | Plan | Schedule | Retention |
|---|---|---|---|
| `iic-prd-db` (production) | `iic-prd-gfs` | Daily + Monthly (1st of month) | 30 daily snapshots **and** 24 monthly snapshots (2-year archive) |
| `iic-master-db` (Forgejo/Jenkins/Phase) | `iic-master-daily-30d` | Daily only | 30 daily snapshots |
| `iic-dev-db` (development) | — | — | Not covered (dev, intentionally) |

Additional protections applied to both production and master:
- **Deletion Protection** enabled — the database instance cannot be accidentally deleted.
- **Point-in-time recovery (PITR)** retained at 1 day — allows restore to *any second* within the last 24 hours (the finest-grained recovery, on top of the daily snapshots).
- **Encryption at rest** — already enabled (AWS KMS).

**Recovery capability after this change:**
- Production: restore to any point in the last 24h, any day in the last 30 days, or the 1st of any month for the last 2 years.
- Master: restore to any point in the last 24h, or any day in the last 30 days.

**Effect on billing:** Snapshots are **incremental** (only changed data is stored), so cost is small. For our database sizes (~3.5 GB each), estimated **≈ $0.50–1 / month** total. Deletion protection and PITR are free.

**Status:** ✅ Live and verified (a test backup job completed successfully).

---

### 2.2 Audit Logging — AWS CloudTrail

**What it is:** Records **every API call / action** taken in the AWS account — who did what, when, and from where — and stores it durably in a private S3 bucket.

**Why we needed it:** There was **no CloudTrail trail configured** — we were relying only on the default 90-day "Event History," which is not durable, not exportable, and disappears after 90 days. The DevZilla investigation depended on that fragile 90-day data; going forward we now have a permanent, tamper-evident trail.

**What we configured:**
- Trail `iic-org-trail` — **multi-region** (captures activity in every region).
- **Log-file validation** enabled (cryptographic digests detect tampering).
- Logs delivered to a dedicated bucket `iic-cloudtrail-logs-891377382138` with **public access blocked** and **versioning enabled**.

**Effect on billing:** The **first trail of management events is free**. The only charge is S3 storage of the logs, which grows slowly — estimated **≈ $0–0.50 / month**. (Optional: a lifecycle rule can auto-expire logs after 1 year to cap storage.)

**Status:** ✅ Live — actively logging (`IsLogging = true`).

---

### 2.3 Threat Detection — Amazon GuardDuty

**What it is:** An automated, machine-learning-based threat-detection service. It continuously analyzes account activity (CloudTrail), network traffic (VPC flow logs), and DNS queries to flag suspicious behaviour — e.g., logins/API calls from unusual locations, credential misuse, or crypto-mining activity.

**Why we needed it:** There was no automated threat detection. GuardDuty would automatically flag exactly the kind of unusual-location intrusion seen in the DevZilla incident — in real time, rather than reconstructed weeks later.

**What we configured:**
- GuardDuty detector enabled in `ap-south-1`, findings published every 15 minutes.

**Effect on billing:** **Free for the first 30 days.** After that, usage-based (driven mainly by network-traffic volume). For an environment our size, estimated **≈ $5–15 / month** — this is the **single largest item** in the added cost. The GuardDuty console shows a live projected cost during the free trial, so the exact figure can be confirmed within a week and the service disabled if desired.

**Status:** ✅ Enabled (in 30-day free trial).

---

### 2.4 Monitoring & Alerts — Amazon CloudWatch + SNS

**What it is:** CloudWatch watches database health metrics; SNS is the notification channel that emails the team when something crosses a threshold.

**Why we needed it:** There were **zero alarms** and **no notification topic** — if a database filled its disk or pegged its CPU, nobody would be alerted.

**What we configured:**
- **SNS topic** `iic-ops-alerts` as the alert channel.
- **4 CloudWatch alarms:**

| Alarm | Trigger |
|---|---|
| `RDS-iic-prd-db-LowStorage` | Free storage < 2 GB on production |
| `RDS-iic-prd-db-HighCPU` | CPU > 80% for 15 min on production |
| `RDS-iic-master-db-LowStorage` | Free storage < 2 GB on master |
| `RDS-iic-master-db-HighCPU` | CPU > 80% for 15 min on master |

- **Email recipients:** `ysnikhil.ai@gmail.com` (confirmed ✅), `jgd@iskconnoida.org` (pending — see §4).

**Effect on billing:** **$0.** The first 10 CloudWatch alarms are free (we have 4), and email notifications are free.

**Status:** ✅ Live and alerting to the confirmed recipient.

---

### 2.5 Account Access Control (IAM) — Posture Review

**What it is:** A review of who can access the account and how well those logins are protected.

**Findings (verified via IAM credential report):**
- **Root account:** ✅ MFA enabled, **zero** access keys — correctly secured.
- **Admin user `ysnikhil`:** ✅ MFA enabled.
- **DevZilla suspect account `TempUser@isknoi`:** no longer present in IAM (appears already removed).

**Effect on billing:** None. IAM and MFA are free.

**Status:** ✅ Reviewed. Some user-level cleanup remains (see §5).

---

## 3. Cost Summary

| Feature | Monthly cost (est.) | Notes |
|---|---|---|
| AWS Backup (snapshots) | $0.50 – $1 | Incremental; small DBs |
| CloudTrail | $0 – $0.50 | First trail free; only S3 storage |
| **GuardDuty** | **$5 – $15** | Free for 30 days; main cost driver |
| CloudWatch alarms + SNS | $0 | Under free-tier limits |
| Deletion protection / PITR / MFA / IAM | $0 | Free |
| **Total added** | **≈ $6 – $16 / month** | ≈ ₹500 – ₹1,350 / month |

> The exact GuardDuty figure will be confirmed during its 30-day free trial via the GuardDuty → Usage console.

---

## 4. Before vs After

| Area | Before | After |
|---|---|---|
| Production backups | 7-day window only | 30 daily + 24 monthly + 1-day PITR |
| Master DB backups | 1-day window only | 30 daily + 1-day PITR |
| Accidental-deletion protection | Off (master) | On (prod + master) |
| Audit trail | None (90-day default only) | Permanent, multi-region, tamper-evident |
| Threat detection | None | GuardDuty active |
| Health alarms | None | 4 alarms emailing the team |
| Notification channel | None | SNS topic with email recipients |

---

## 5. Outstanding Items (being handled by the team)

| # | Item | Owner | Priority |
|---|---|---|---|
| 1 | **MFA for 4 human users** (`abhishek.vishwakarma`, `dhruv.krishna.vaid`, `krishnaJadhav`, `neelmani.upadhyay`) — console logins currently without MFA | Team | **High** |
| 2 | **Confirm `jgd@iskconnoida.org`** SNS subscription — currently pending; the confirmation email is not reaching that mailbox (likely the alias is not set up / mis-routed on the domain). Needs the mailbox verified, then re-subscribe + confirm. | Tech | Medium |
| 3 | **Rotate old service-account access keys** (`iskconnoida-user` ~18 mo, `iic-ses-smtp-user` ~14 mo, others) using the safe two-key method | Team | Medium |
| 4 | **Remove dormant IAM user** `iic-stage-user` (no active credentials) | Team | Low |

**Optional future enhancements (not yet done):**
- CloudTrail → CloudWatch security alarms (alert on root login, CloudTrail being disabled, security-group changes) — real-time intrusion warning. Cost: free.
- Billing/cost-anomaly alarm (catches cost spikes / crypto-mining). Cost: free.
- EC2 status-check alarms for the production app host and master server. Cost: free.
- Multi-AZ for the production database (automatic failover for high availability). Cost: roughly doubles the DB instance price — a budget decision.

---

## 6. Resource Reference (Appendix)

| Type | Name / ID |
|---|---|
| Backup vault | `iic-prd-vault` |
| Backup plan (prod) | `iic-prd-gfs` (Daily-30d + Monthly-24m) |
| Backup plan (master) | `iic-master-daily-30d` (Daily-30d) |
| Backup service role | `AWSBackupDefaultServiceRole` |
| CloudTrail trail | `iic-org-trail` (multi-region, validated) |
| CloudTrail log bucket | `iic-cloudtrail-logs-891377382138` |
| GuardDuty detector | `34cf76e01c8bf5842d3e30de9d8de99a` |
| SNS alert topic | `iic-ops-alerts` |
| CloudWatch alarms | `RDS-{iic-prd-db,iic-master-db}-{LowStorage,HighCPU}` |
| Databases | `iic-prd-db` (prod), `iic-master-db` (master), `iic-dev-db` (dev) |

---

## 7. Technical Appendix — Health-Check Commands

These are **read-only verification commands** to confirm each feature is healthy (or detect if it has stopped). Run them in **AWS CloudShell** (region `ap-south-1`). Each block notes what a healthy result looks like.

### A. Database Backups

```bash
# Backup plans exist
aws backup list-backup-plans --region ap-south-1 \
  --query 'BackupPlansList[].{Plan:BackupPlanName,Id:BackupPlanId}' --output table

# Recent backup jobs succeeded (and check for failures)
aws backup list-backup-jobs --region ap-south-1 --by-state COMPLETED \
  --query 'BackupJobs[:5].{Resource:ResourceArn,State:State,Completed:CompletionDate}' --output table
aws backup list-backup-jobs --region ap-south-1 --by-state FAILED \
  --query 'BackupJobs[:5].{Resource:ResourceArn,Msg:StatusMessage}' --output table

# RDS retention + deletion protection on both DBs
for DB in iic-prd-db iic-master-db; do
  aws rds describe-db-instances --region ap-south-1 --db-instance-identifier $DB \
    --query 'DBInstances[].{DB:DBInstanceIdentifier,RetentionDays:BackupRetentionPeriod,DeleteProtect:DeletionProtection,LatestRestore:LatestRestorableTime}' --output table
done
```
✅ **Healthy when:** both plans listed; recent jobs show `COMPLETED` and the FAILED list is empty; `DeleteProtect = true`; `RetentionDays ≥ 1`; `LatestRestore` is within the last few minutes.

### B. CloudTrail (Audit Logging)

```bash
aws cloudtrail get-trail-status --name iic-org-trail --region ap-south-1 \
  --query '{IsLogging:IsLogging,LatestDelivery:LatestDeliveryTime,LatestError:LatestDeliveryError}' --output table
aws cloudtrail describe-trails --region ap-south-1 \
  --query 'trailList[?Name==`iic-org-trail`].{MultiRegion:IsMultiRegionTrail,Validation:LogFileValidationEnabled,Bucket:S3BucketName}' --output table
```
✅ **Healthy when:** `IsLogging = True`, `LatestError` is empty, `LatestDelivery` is recent, `MultiRegion = True`, `Validation = True`.
🔴 **Stopped if:** `IsLogging = False` or the trail is missing.

### C. GuardDuty (Threat Detection)

```bash
DET=$(aws guardduty list-detectors --region ap-south-1 --query 'DetectorIds[0]' --output text)
aws guardduty get-detector --region ap-south-1 --detector-id $DET \
  --query '{Status:Status,Frequency:FindingPublishingFrequency}' --output table
# Count of open findings (0 is normal/healthy)
aws guardduty list-findings --region ap-south-1 --detector-id $DET --query 'length(FindingIds)' --output text
```
✅ **Healthy when:** `Status = ENABLED`.
🔴 **Stopped if:** `DET` is empty or `Status` is not `ENABLED`.
⚠️ Investigate any open findings (the count > 0).

### D. CloudWatch Alarms (Monitoring)

```bash
aws cloudwatch describe-alarms --region ap-south-1 \
  --query 'MetricAlarms[].{Alarm:AlarmName,State:StateValue}' --output table
```
✅ **Healthy when:** all 4 alarms exist and show `OK`.
⚠️ `ALARM` = the condition is breached (low storage / high CPU) — act on it.
ℹ️ `INSUFFICIENT_DATA` is normal only briefly after (re)creation.

### E. SNS Alert Delivery

```bash
aws sns list-subscriptions-by-topic --region ap-south-1 \
  --topic-arn arn:aws:sns:ap-south-1:891377382138:iic-ops-alerts \
  --query 'Subscriptions[].{Endpoint:Endpoint,Confirmed:SubscriptionArn}' --output table
```
✅ **Healthy when:** each recipient shows a long `arn:aws:sns:...` value.
🔴 **Not receiving alerts if:** an endpoint shows `PendingConfirmation` (link not clicked) or `Deleted` (unsubscribed).

### F. IAM / MFA

```bash
# Root account: must be MFA on, zero access keys
aws iam get-account-summary \
  --query 'SummaryMap.{RootMFA:AccountMFAEnabled,RootKeys:AccountAccessKeysPresent}' --output table

# Which console users have MFA (mfa column should be true for every user with a password)
aws iam get-credential-report --query Content --output text | base64 -d \
  | awk -F, 'NR==1{print "user,password,mfa"} NR>1 && $4=="true"{print $1","$4","$8}' | column -t -s,
```
✅ **Healthy when:** `RootMFA = 1`, `RootKeys = 0`, and `mfa = true` for every user that has a password.

---

> **Handling note:** This document references internal infrastructure identifiers (account ID, resource names, alert emails). It contains **no passwords or access keys**, but should be shared on a need-to-know basis and is recommended **not** to be committed to the shared application repository.
