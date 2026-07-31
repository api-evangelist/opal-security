---
name: sox-uar
description: Execute a SOX-compliant User Access Review (UAR) campaign end-to-end for GRC and enterprise security teams. Use when running a quarterly access recertification, preparing for a SOX ITGC audit, scoping a new system for access review, chasing down reviewer signoff, processing keep/revoke decisions, generating audit evidence packages, or remediating access certification exceptions. Triggers on phrases like "run the UAR", "quarterly access review", "access recertification", "SOX access review", "ITGC evidence", "audit prep", "reviewer chase", or "revocation SLA".
when_to_use: The quarter has turned and the ITGC control owner needs a UAR campaign opened. Internal audit is requesting evidence of a prior period review. A newly in-scope system needs to be added to the UAR rotation. A reviewer is stuck and needs guidance. Revocations are sitting past SLA and need escalation.
allowed-tools: Read Grep Glob Bash(gh *)
---

# SOX User Access Review (UAR)

This skill runs a SOX-compliant User Access Review for in-scope systems. It assumes a GRC team owns the control narrative and evidence, and an enterprise security / IAM team executes the campaign and remediation. It is written for practitioners — it does not re-explain SOX, ITGC, or what a UAR is.

The goal of a UAR is not to "review access." The goal is to produce **defensible, complete, timely evidence** that every user's access to every in-scope system was independently certified by an appropriate reviewer, and that revocations happened within SLA. Everything else serves that.

## Phases

A UAR campaign has six phases. Do not skip ahead. Each phase has a gate that must be cleared before the next one starts.

1. [Scope & plan](#1-scope--plan)
2. [Collect & reconcile](#2-collect--reconcile)
3. [Launch campaign](#3-launch-campaign)
4. [Reviewer execution](#4-reviewer-execution)
5. [Remediation](#5-remediation)
6. [Evidence & signoff](#6-evidence--signoff)

---

## 1. Scope & plan

**Gate to clear:** GRC-signed scoping memo with system list, reviewer mapping, and deadlines.

### In-scope systems
Pull the current SOX ITGC matrix from GRC. In-scope systems are anything that touches financial reporting — directly (GL, ERP, revenue systems, close/consolidation tools) or indirectly (IdP, cloud infra hosting in-scope apps, ticketing for change mgmt, code repos for in-scope apps). Do not rely on last quarter's list. New systems get added mid-year; acquired entities bring new systems; retired systems need to come off.

For each system, confirm with GRC:
- Control ID (e.g., ITGC-AC-04)
- Review frequency (quarterly is typical; some controls are semi-annual or annual)
- Reviewer type: **manager** (reviews direct reports' access), **resource owner** (reviews all users of a system), or **both** (dual review — required for privileged access on sensitive systems)
- Evidence format auditors expect (reviewer name + decision + timestamp + rationale for "keep" on privileged)

### Deadlines (work backwards from auditor deadline)
- **T-0:** Auditor evidence due
- **T-5 business days:** GRC signoff memo complete
- **T-10:** Remediation SLA expires (all revocations executed)
- **T-17:** Reviewer decisions due (7 business days to review)
- **T-20:** Campaign launched, reviewers notified
- **T-25:** Data reconciled, population locked
- **T-30:** Scoping complete, kickoff

### Reviewer mapping
For each system, build a reviewer map: `{user, entitlement, system} → reviewer`. Source of truth for manager chain is HRIS — never trust IdP groups or directory info that may be stale. For resource owners, confirm in writing that the listed owner is still the owner (ownership changes are a top audit finding).

**Conflict check:** a reviewer cannot review their own access. Build this check into the routing logic, not as a manual step.

---

## 2. Collect & reconcile

**Gate to clear:** Locked, reconciled population with sign-off from system owners that the user/entitlement extract is complete and accurate as of the review date.

### Extract
Pull from each in-scope system:
- User identifier (email preferred — map to HRIS employee ID)
- Entitlement (role, group, permission set — whatever the system exposes)
- Account type (human, service, shared, break-glass)
- Last login / last used (if the system supports it)
- Grant date, granted-by (if available)

Freeze the extract to a specific timestamp. Auditors test completeness by re-pulling the same extract on the same date; drift between your population and the re-pull is a finding.

### Reconcile against HRIS
Run three reconciliations before launch:

1. **Terminations:** any user in the extract whose HRIS status is "terminated" is a P0 finding — they should not have access. Open a severity-1 incident; do not let this sit in the UAR queue.
2. **Transfers:** users whose role/department changed since last review need their reviewer remapped and entitlements re-evaluated (not just recertified).
3. **Orphans:** accounts with no matching HRIS record. Investigate before launch — these are either service accounts (flag separately) or ghost accounts (incident).

### Non-human identities
Service accounts, shared accounts, and machine identities have separate review paths. Do not route them to managers — managers cannot meaningfully certify a service account. Route them to the technical owner of the account (whoever is listed in the secrets manager / IAM inventory), and require a business justification for every "keep." Unreviewed NHIs are an increasingly common audit finding, especially post-2024 guidance.

### Privileged access
Flag admin / privileged entitlements separately. These get:
- Dual review (resource owner + security)
- Mandatory written justification for "keep"
- Shorter remediation SLA (typically 3 business days, not 7)

---

## 3. Launch campaign

**Gate to clear:** Campaign live, reviewers notified, acknowledgment tracking started.

### Communications
Send reviewer notifications with:
- What they're reviewing (system, number of items)
- Deadline (absolute date, with business-day count)
- Reviewer guide link (one page, not a 40-slide deck)
- Escalation path ("if you can't reach the user, contact X")
- Consequences of non-response (this is a control failure — say so plainly)

Send a copy to each reviewer's manager. This is not to shame anyone; it's so that when a reviewer is OOO, the manager knows a decision is pending.

### Reviewer guide essentials
A reviewer needs to know exactly four things:
1. How to tell if access is still needed (job duties, last login, project involvement)
2. What to do for each option (keep / revoke / modify / escalate)
3. When "keep" requires written justification (always, for privileged)
4. Who to ask if they're unsure

Anything longer than one page gets skimmed and the control effectiveness drops.

---

## 4. Reviewer execution

**Gate to clear:** 100% of items have a recorded decision from the correct reviewer, within the deadline window, with timestamps.

### SLA tracking
Track acknowledgment (reviewer opened the campaign) separately from completion. A reviewer who hasn't acknowledged at T-minus-3 days needs a personal nudge, not another automated email.

### Escalation ladder
- Day 3 of silence: automated reminder
- Day 5: personal message from security/GRC
- Day 6: manager of reviewer notified
- Day 7 (deadline): control owner escalates to reviewer's VP
- Past deadline: documented control exception, must be approved by the control owner and logged in the exception register

Reviewers skipping a UAR is the single most common SOX deficiency. Do not let it slide because it is socially awkward. The deficiency costs more than the awkwardness.

### Decision quality
Spot-check 10% of "keep" decisions for reasonableness. If a reviewer kept everything in under 90 seconds for 200 items, that's rubber-stamping and the review is not effective. Re-run those items with a different reviewer or add a secondary review step.

Quality spot-checks are a control over a control. Auditors like to see them. GRC should own this, not the team being reviewed.

---

## 5. Remediation

**Gate to clear:** All revocations executed within SLA, with system-generated evidence of removal.

### Ticket the revocation
Every "revoke" decision generates a ticket to IAM / security with:
- User, entitlement, system
- Decision timestamp and reviewer
- Target completion date (7 BD standard, 3 BD privileged)
- Verification method (how we will prove it was removed)

### Verification
Do not trust "ticket closed" as evidence of revocation. Verify from the target system: re-pull the user's entitlements post-revocation and confirm the specific entitlement is gone. Auditors will re-pull independently — you want your evidence to match theirs.

### Exception register
If a revocation cannot be completed in SLA (system outage, business-critical dependency, etc.), it goes in the exception register with:
- Reason
- Compensating control
- Target completion date
- Approver (must be control owner or above)

An exception is not a failure — an exception without documentation is.

---

## 6. Evidence & signoff

**Gate to clear:** Audit package delivered to GRC, control owner memo signed.

### Audit package contents
For each system in the review:
1. **Control narrative** — what the control is, how it's designed to work
2. **Population list** — the frozen extract from Phase 2, with timestamp
3. **Reviewer assignments** — who reviewed what, with conflict-check evidence
4. **Decisions log** — every keep/revoke/modify with reviewer, timestamp, rationale (for privileged keeps)
5. **Revocation evidence** — tickets + system-generated confirmation of removal
6. **Exception log** — with approvals
7. **Quality spot-check results** — Phase 4 re-reviews
8. **Control owner signoff memo** — one page: scope, completeness statement, exceptions, signoff

### Completeness statement
The signoff memo must explicitly state that (a) the population was complete as of the review date, (b) all items received a decision from an appropriate independent reviewer, (c) revocations were completed or formally excepted, and (d) evidence is retained per retention policy. Generic boilerplate will get kicked back by auditors.

### Handoff to GRC
GRC owns the control and will present to internal/external audit. Security/IAM's job ends at a clean, complete package delivered on time. If GRC has questions during audit testing, respond within one business day — auditor questions age fast.

---

## Common pitfalls (learned the hard way)

These are the failure modes that turn a routine UAR into a deficiency finding:

- **Stale reviewer lists.** Resource owner left the company three months ago; the UAR routes to a deactivated mailbox. Refresh ownership every quarter, not annually.
- **Population drift.** Extract was pulled on Monday, campaign launched on Friday. Users provisioned in between are missing. Freeze the extract at campaign launch, not earlier.
- **Service accounts in manager queues.** Managers cannot certify service accounts. Every time this happens, the control is deficient for that account.
- **"Temporary" elevated access.** Break-glass and just-in-time grants often don't appear in standing entitlement extracts. Confirm with the access platform that time-bound grants are included in the review population or covered by a separate control.
- **Reviewer = reviewee.** A director reviewing their own VP-level access. Conflict check must be automated.
- **Revocation SLA drift.** Revocation tickets that sit open past SLA with no exception logged. This is a control failure regardless of whether the user did anything with the access.
- **Evidence that doesn't tie out.** The decision log says 1,247 items reviewed; the revocation log says 312 revocations; the population extract has 1,249 users. Reconcile numbers end-to-end before handoff.
- **Treating NHIs like an afterthought.** In 2025–2026 guidance, auditors are specifically probing NHI review processes. If yours is "we don't really review those," the finding is already written.

---

## Outputs this skill produces

When invoked, this skill drives toward the following deliverables in order:

1. Scoping memo (Phase 1)
2. Reconciled population extract (Phase 2)
3. Live campaign with reviewer assignments (Phase 3)
4. Decisions log with SLA tracking (Phase 4)
5. Revocation tickets + verification evidence (Phase 5)
6. Full audit package + control owner signoff (Phase 6)

If you're brought in mid-campaign, identify the current phase from the artifacts that exist, and resume from the next incomplete gate. Do not restart.

---

## When this skill should not run

- **Ad-hoc access reviews** that aren't tied to a SOX control — use a lighter-weight recertification flow, not the full evidentiary package.
- **Non-SOX regulatory reviews** (HIPAA, SOC 2, ISO 27001) — the phases are similar but the evidence requirements and control language differ. Do not reuse the signoff memo template.
- **Joiner-mover-leaver spot checks** — this is a different control. UAR certifies standing access; JML certifies access changes at employment transitions.

If the request is outside SOX UAR scope, say so and point to the right process rather than forcing this skill to cover it.
