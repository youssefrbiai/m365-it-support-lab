TKT-007-offboarding-departure.md

```markdown
# TKT-007 — Employee Departure / Offboarding (Robert Klein)

**Date:** 26/09/2026
**Priority:** 🔥 HIGH (same-day, security-sensitive, HR conflict risk)
**SLA Target:** Access removed BEFORE 16:00
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## Request

HR (Sofia Ramos) submitted an urgent departure notice:

> "Robert Klein in Sales was let go this morning. His manager says he was
> 'not happy about the decision.' He leaves the office at 4 PM today.
> Please handle his access immediately. Also — his manager needs access
> to his client emails and the Sales reports he owned."

**Three separate problems in one request:**
1. **Security:** cut access before departure — and before a disgruntled
   employee has hours of unsupervised active access
2. **Data:** manager needs his client emails and files
3. **Cleanup:** what happens to account, license, device?

---

## Triage Decisions

**Q: Disable now or at 4 PM?**
→ **DISABLE IMMEDIATELY.** The 4 PM deadline is the *last* moment, never
the target. Between 9 AM and 4 PM, an unhappy employee with active access
can email files to personal accounts, delete documents, or sabotage data.
The nuanced case: if he legitimately needed email for a handover meeting,
the manager can supervise — but the account is never left enabled on hope.

**Q: Who else must be informed before touching the account?**
→ **His manager** — he depends on the outcome (mailbox access, handover
tasks) and must coordinate any final legitimate needs.

---

## Execution — The Offboarding Sequence

```powershell
Import-Module ActiveDirectory

# (Robert's account was created this morning: rklein, Sales, SG-Sales member)

# A. Disable IMMEDIATELY — the security cut (blocks all new logons)
Disable-ADAccount -Identity "rklein"

# B. Remove group memberships — access must not linger on a dead account
Remove-ADGroupMember -Identity "SG-Sales" -Members "rklein" -Confirm:$false

# C. Reset password one final time — kills cached credentials
Set-ADAccountPassword -Identity "rklein" -Reset `
    -NewPassword (ConvertTo-SecureString "X91qZv2#LmWp4$eR@Q@FRfa23" -AsPlainText -Force)

# D. Verify full state
Get-ADUser -Identity "rklein" -Properties Enabled, MemberOf, PasswordLastSet |
    Select-Object Name, Enabled, MemberOf, PasswordLastSet
# Result: Robert Klein — Enabled: False — MemberOf: {} — fresh PasswordLastSet ✅
```

---

## Why Disable AND Reset? (Defense in Depth)

Each control covers a hole the other leaves:

| Control | What it blocks | What it does NOT block |
|---|---|---|
| **Disable** | All NEW logon attempts | Already-open sessions reconnecting; devices using cached credentials |
| **Password reset** | Cached credentials (phone, home PC), reconnecting sessions | Brand-new logon attempts (already covered by disable) |

Two cheap actions together = full coverage. One alone = a gap.

---

## The Data Question: Mailbox & Files

**Mailbox → convert to a SHARED MAILBOX (do NOT delete):**

- Manager keeps full access to client emails
- A shared mailbox requires **NO license — $0/month** (a normal licensed
  mailbox keeps costing money for a user who is gone)
- Data is preserved indefinitely instead of vanishing after deletion

**Account deletion timing:**

- Minimum **30 days** (the M365 Recycle Bin / soft-delete window)
- The decision belongs to **HR/legal — never IT.** IT executes; the
  business decides when data may be destroyed (notice periods, legal
  holds, audits run in weeks/months, not hours)

---

## The Forgotten Step: The Device

Before Robert walks out with a **company laptop**:

1. **Device returned** — verify physically before he leaves
2. **Check sync completed** — OneDrive/backup; unsynced company files
   live only on that laptop
3. **Reimage before reissue** — browser-saved passwords and local caches
   survive user-account deletion; wiping the user is not wiping the device

**If he were remote and kept the laptop:** the control is a
**REMOTE WIPE via Microsoft Intune** — a command that makes the laptop
wipe itself on its next internet connection. Plain on-prem AD **cannot**
do this; Intune/Entra device management is what adds that capability.

---

## M365 Offboarding Mapping (Standard Sequence)

| M365 Offboarding Step | Why |
|---|---|
| **Block sign-in** (immediately) | Cuts access instantly |
| **Convert mailbox to shared** | Manager keeps client email access; $0 extra license |
| **Remove license** | Frees it for reassignment; data retained during the 30-day window |
| **Grant manager OneDrive access** (BEFORE license removal) | Files survive the user — access must be granted first or you may lock yourself out |
| **Delete user** (after HR/legal hold) | Goes to Recycle Bin for **30 days**, then permanent |

---

## Notes / Dead Ends (Troubleshooting Artifacts)

- ⚠️ **Real error hit and fixed:** first password reset attempt failed —
  `ADPasswordComplexityException: The password does not meet the length,
  complexity, or history requirement of the domain` — the initial temp
  password was rejected (domain enforces MinPasswordLength=14,
  PasswordHistoryCount=24 — see TKT-001 policy). Retried with a longer
  random password (`...@Q@FRfa23`) → succeeded. Proves the domain policy
  is enforced in practice, and demonstrates the read-error → adjust →
  retry cycle.
- Initial draft of deletion timing ("wait 3 hours") was wrong — corrected
  to 30 days minimum with HR/legal as the decision owner.

---

## Lessons Learned

1. **Offboarding is a security event, not paperwork** — the gap between
   "let go this morning" and "disabled" is the attacker's window.
2. **Disable + password reset together** — defense in depth; each covers
   the other's blind spot (new logons vs cached credentials/sessions).
3. **Shared mailbox conversion** is the standard answer for departed-user
   email: manager access + $0 license + preserved data.
4. **IT never decides deletion timing** — 30-day soft-delete minimum,
   HR/legal owns the decision.
5. **The device outlives the user account** — return, sync-check, reimage;
   remote wipe (Intune) for devices you can't physically collect.
6. **Sequence matters in M365:** grant manager OneDrive access BEFORE
   removing the license, or the data path is cut.

---
