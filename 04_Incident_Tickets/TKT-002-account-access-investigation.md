TKT-002-account-access-investigation.md

```markdown
# TKT-002 — Account Access Investigation (Maria Gonzalez)

**Date:** 25/09/2026
**Priority:** High (business impact — client presentation at 2:00 PM)
**SLA Target:** 30 minutes
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## Symptom

User Maria Gonzalez (Sales) reported she could not log in to her computer.
Her words: *"It says my account is locked or something?? I typed my password
like a hundred times."* She needed access urgently for a 2 PM client
presentation.

**Important:** The user's self-diagnosis was "locked." Self-diagnosis is a
symptom, not evidence — it must always be verified against the system.

---

## Questions Asked (Tier 1 Triage)

1. *"Can you read me the EXACT message on the screen, word for word?"*
   — The exact wording distinguishes locked vs disabled vs expired password.
2. *"Is anyone else in Sales having this problem, or just you?"*
   — One user = account problem; many users = outage/network problem.
3. *"Did you change your password recently — and when did it last work?"*
   — Recent resets/expiry are a top cause of "locked out" tickets.

> Note: First draft of question 2 was vague ("did you change... internet
> working?"). Revised during triage to a proper scoping question.

---

## Diagnosis

Ran full account forensics on the DC:

```powershell
Get-ADUser -Identity "mgonzalez" -Properties * |
    Select-Object Name, Enabled, LockedOut, PasswordExpired,
                  LastBadPasswordAttempt, BadLogonCount,
                  AccountLockoutTime, LastLogonDate
```

**Results:**

| Property | Result | Meaning |
|---|---|---|
| Enabled | **False** | 🚨 Root cause — account is disabled |
| LockedOut | False | NOT a lockout (user's self-diagnosis was wrong) |
| PasswordExpired | False | Password expiry ruled out |
| LastBadPasswordAttempt | (blank) | No failed attempts ever recorded |
| BadLogonCount | 0 | "Typed it 100 times" — yet zero counted |
| AccountLockoutTime | (blank) | Account has never been locked |
| LastLogonDate | (blank) | User has NEVER successfully logged on |

---

## Root Cause

The account was **DISABLED** (an administrative action), not locked.

Evidence chain:
- `Enabled: False` — the account is switched off at the admin level.
- `BadLogonCount: 0` proves wrong passwords were never even counted:
  a disabled account rejects the sign-in **before** password validation.
  The password was never the problem.
- `LastLogonDate: (blank)` shows the account never had a successful
  logon, which contradicts the user's implied history of working access.

---

## Why a Lockout Was Impossible in This Domain

Domain policy (from `Get-ADDefaultDomainPasswordPolicy`, TKT-001):

```
LockoutThreshold = 0
```

`LockoutThreshold = 0` does NOT mean "lock after one bad attempt" — it
means **lockout is disabled domain-wide**. Maria's 100 wrong attempts
would never lock the account regardless.

Real-world context: many organizations deliberately set threshold = 0
(or very high) to prevent attackers from weaponizing lockouts — a
denial-of-service where an attacker intentionally mass-locks employee
accounts by entering wrong passwords. Trade-off: bad-password blocking
must be replaced by other controls (MFA, monitoring).

---

## Fix

```powershell
Enable-ADAccount -Identity "mgonzalez"
```

**Verification:**

```powershell
Get-ADUser -Identity "mgonzalez" -Properties Enabled, LockedOut |
    Select-Object Name, Enabled, LockedOut
# Result: Enabled = True, LockedOut = False ✅

Get-ADGroupMember -Identity "SG-Sales"
# Result: mgonzalez present — group membership intact ✅
```

**Resolved within SLA.** User made her 2 PM presentation.

---

## Key Concept: Locked vs Disabled

| | Locked | Disabled |
|---|---|---|
| Cause | Too many failed password attempts (automatic) | Deliberate administrative action |
| Duration | Temporary — clears after lockout duration or admin unlock | Permanent until an admin enables it |
| Password still valid? | Yes | Yes — but never checked; rejection happens first |

**Which we simulated:** Disabled (`Disable-ADAccount` → `Enable-ADAccount`).

**M365 / Entra ID equivalents:**
- Admin Center → Users → Active users → select user → check
  **Block sign-in** status (the cloud version of `Enabled`).
- Entra ID → **Sign-in logs** → each failed attempt shows an exact
  failure reason (`UserAccountDisabled` vs `LockedOut` vs
  `InvalidPassword`) — the cloud version of this forensic command.

---

## Notes / Dead Ends

- Initial triage question 2 was vague ("did you change... is your internet
  working?") — revised to a scoping question during the call.
- User said password was typed "a hundred times," but BadLogonCount = 0.
  Investigating this contradiction (not just accepting the user's story)
  is what identified the disabled account.

---

## Lessons Learned

1. **Never trust the user's terminology — trust the system's evidence.**
   "Locked" and "disabled" feel identical to users but are different problems.
2. **A disabled account rejects before password validation** — that's why
   bad-password counters stay at 0.
3. **Read the whole picture:** LastLogonDate blank + BadLogonCount 0 +
   Enabled False tells a complete story in one screen of output.

---

*Methodology: Identify → Isolate → Test → Root Cause → Fix → Verify → Document*
*Related: TKT-001 (user lifecycle script failure — established domain password policy and AD module usage)*
```