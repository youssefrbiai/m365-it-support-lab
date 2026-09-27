TKT-009-license-revocation-access-loss.md

```markdown
# TKT-009 — License Revocation Causing Total Service Loss (Maria Gonzalez)

**Date:** 27/09/2026
**Priority:** High
**SLA Target:** 30 minutes
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** M365 scenario (knowledge-based — no tenant available; on-prem lab has no licensing system)

---

## Symptom

Maria Gonzalez (Sales) reported THREE services failing simultaneously:

- Outlook: "no connection"
- Teams: signed out, cannot rejoin
- OneDrive: "storage read-only"

Her words: *"I didn't touch anything. Yesterday everything worked."*
She also noted the unfairness: *"Aisha started AFTER me and her stuff
works fine!"*

**Admin-side view:**
- Company holds 20 × Microsoft 365 Business Premium licenses
- Finance memo: "unused/inactive licenses will be reclaimed this week"
- Maria's license was **unassigned at 11 PM last night**
- Aisha's license: assigned, active

---

## Diagnosis — The Three-Products-At-Once Pattern

**Key insight:** Outlook + Teams + OneDrive failing simultaneously is not
three problems — it is **ONE license** manifesting three symptoms. In
M365, a single license (here: Business Premium) unlocks every service at
once; remove it, and everything gated behind it drops together.

**The Aisha clue:** Aisha was onboarded LAST WEEK (TKT-006) with a freshly
assigned license. Maria was an established account. The sweep targeted
accounts that *looked* inactive — new accounts were safe, older quiet
ones were harvested. The issue is on her **account**, not her computer.

**Was it an error or a process?** It was a **deliberate but flawed
process** — finance acted on their memo but never verified actual usage
(last sign-in) before classifying a license as "unused." The fix is not
"undo a typo" — it is **overriding a cost decision with usage evidence.**

---

## The Fix — Admin Center Path

> **Microsoft 365 Admin Center → Users → Active users →
> select Maria Gonzalez → Licenses and apps → check
> Microsoft 365 Business Premium → Save changes**

### ⚠️ The Budget Trap (Check BEFORE Assigning)

Assigning a license = spending money. Before restoring:

1. **Is there a free (unassigned) license?** → If yes: assign + notify
   finance with usage evidence.
2. **If all 20 are allocated** → restoring Maria's license means taking
   one from someone else or purchasing another — that is a
   **manager/finance decision**, escalated with evidence. Tier 1 does
   not silently spend the company's money.

In this case: a free license existed → restored → services returned.

---

## What Happens to Data When a License Is Removed

- **Nothing is deleted immediately.** Mailbox and OneDrive files are
  retained — Maria's OneDrive "read-only" banner was the clue: data
  present, access frozen.
- **Removed → re-added within 30 days:** everything comes back intact.
- **Removed for more than 30 days:** mailbox and OneDrive data are
  **permanently deleted.**

### Order of Operations (Never Break This)

Before ANY license removal (TKT-007 lesson):

1. **Convert mailbox to shared** (manager keeps client email, $0 license)
2. **Grant manager OneDrive access** — BEFORE the license disappears,
   or the data path is cut
3. Only then: remove license

---

## Prevention — Making This Impossible to Recur

1. **Process rule proposed:** no license is reclaimed without
   (a) IT confirming "no sign-in for X days" from last-sign-in data,
   (b) the user's manager approving, and
   (c) a documented reclaim list (account, reason, evidence) — so every
   removal is traceable and reversible.
2. **Group-based licensing (Entra ID):** if SG-Sales had group-based
   licensing configured, every member automatically holds a license —
   new hires get one on joining, and a manual sweep cannot pluck
   individuals out. This is the systemic fix for this entire ticket class.

---

## Communication — Callback Script

> *"Hello Ms. Maria, it's IT support again. Good news — everything is
> fine from your side, no worries, and nothing was lost. A licensing
> change on the account removed your subscription overnight, which is
> why Outlook, Teams, and OneDrive all stopped at once. We've restored
> it — give it a few minutes and everything will come back exactly as
> you left it, all your emails and files included. About Aisha — her
> account was set up just last week, so it was automatically on the
> active list; this had nothing to do with you or your work."*

**Techniques used:** data-safety reassurance FIRST (her biggest fear),
plain-English cause, honest timing, and the fairness line — without
making another department the villain in front of staff (internally,
finance was briefed with usage evidence; externally, neutral wording).

---

## M365 Concepts Covered

| Concept | Answer |
|---|---|
| One license → all services | Business Premium gates Outlook, Teams, OneDrive together |
| License removed | Data retained, access frozen (read-only), 30-day grace |
| License gone >30 days | Mailbox + OneDrive permanently deleted |
| Before removing any license | Shared mailbox conversion + OneDrive access granted first |
| Systemic fix | **Group-based licensing** in Entra ID |
| Spending decision | Assigning/reclaiming licenses = finance/manager call, with IT evidence |

---

## Notes / Dead Ends

- Initial read of the incident as a "finance mistake (accident)" was
  corrected: the memo shows a deliberate process with a flawed judgment
  step (no usage check) — which changes the fix from "undo an error" to
  "override with evidence + escalate the process."
- No tenant available: admin-center steps documented as walkthrough,
  consistent with this project's constraint (on-prem DC is the hands-on
  environment; M365 flows are knowledge-based).

---

## Lessons Learned

1. **Multiple services failing at once = suspect the license first** —
   one subscription gates many products.
2. **Read-only OneDrive = data retained, access frozen** — reassure the
   user their data exists before anything else.
3. **30-day grace window** on license removal — but never rely on it;
   follow the order of operations before any removal.
4. **Assigning a license is a financial action** — check availability,
   escalate if it means taking or buying.
5. **Group-based licensing makes manual sweeps obsolete** — automate
   entitlement, don't audit it by hand.
6. **Communication rule:** own the fix, protect the user's peace of mind,
   and never air interdepartmental conflict in front of staff.

---
