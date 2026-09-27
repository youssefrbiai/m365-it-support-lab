TKT-010-monday-storm-final-exam.md

```markdown
# TKT-010 — The Monday Morning Storm (Final Exam — Multi-Incident Triage)

**Date:** 28/09/2026
**Priority:** Mixed queue — triaged per ticket
**SLA Target:** Per-ticket, triaged by risk
**Status:** ✅ RESOLVED (Ticket D escalated to security team — containment complete)
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## The Scenario — Four Tickets in Three Minutes

**8:45 AM Monday. The queue floods in:**

| # | Ticket | Reporter | Symptom |
|---|---|---|---|
| A | Can't log in at ALL | Maria Gonzalez (Sales) | "Password is right, something's wrong, second time IT breaks my Monday!" (angry, single user) |
| B | Contractor + laptop question | Sofia Ramos (HR) | Contractor starting NEXT Monday needs an account; whatever happened to Robert Klein's laptop? ("no rush") |
| C | Whole team denied | David Chen (Marketing) | ALL FOUR Marketing users get access denied on the shared folder ("fine on Friday!") |
| D | 🚨 Security escalation | Manager (via security team) | Sign-in attempt on Aisha's account at 3 AM from outside the country — AND her phone's Authenticator APPROVED something at 3 AM while she was asleep |

---

## Triage Order: D → C → A → B

1. **D — security event, active damage possible NOW.** The attacker's
   authentication factor was APPROVED — this is not a suspicion anymore,
   the account was entered. Contain before anything else.
2. **C — whole-team outage.** Four users blocked = business-wide impact
   on Marketing, but no security dimension. Fix is likely quick (group-level).
3. **A — single user, login problem.** Feels loud (angry user) but it is
   ONE account with a sign-in issue = lowest risk. The TKT-002 playbook
   applies directly.
4. **B — formality with a deadline.** Nothing is broken today, but "no
   rush" must never become "forgotten" (see Part 4).

**The trap:** Ticket A *feels* most urgent because the user is angry and
it's a repeat complaint — but "loud ≠ urgent." One user + login issue +
no security signal = lowest actual risk in the queue.

**The two-in-one ticket:** B contains two unrelated tasks (contractor
onboarding + Robert's laptop loose end from TKT-007) — it must be split
into two actions, not handled as one.

---

## Pattern Matching — Each Ticket to Its History

| Ticket | Likely cause (from project history) | First action |
|---|---|---|
| A — Maria can't log in | TKT-002 candidates: **disabled** vs **bad password/other** — told apart by the EXACT error text + `Get-ADUser` forensics (Enabled, LockedOut, PasswordExpired, BadLogonCount) | Ask for exact message word-for-word; run account forensics |
| B — contractor + laptop | TKT-006 onboarding checklist + TKT-007 device return/reimage | Confirm contractor details; verify laptop returned, synced, queued for reimage |
| C — whole team denied | TKT-004 logic inverted: ONE user = account problem; **ALL users = shared resource/group problem** | Check group object: members, name, SID |
| D — Aisha 3 AM + approved factor | TKT-008 containment sequence — but worse (see below) | **Contain first:** disable + reset + revoke/re-register MFA |

---

## Ticket D — The Security Event (Priority 1)

### What makes this DIFFERENT from TKT-008

In TKT-008, the MFA factor **blocked** the attacker → *suspected*
compromise, report and move on.

Here, the factor **APPROVED** while Aisha slept → **CONFIRMED
compromise.** Someone authenticated successfully at 3 AM. The account
was entered. Everything after that is incident response, not triage.

### Where did her password go? (Root-cause hypotheses)

Aisha's account was created in TKT-006 with a password only she knew.
For someone to also pass MFA at 3 AM, likely vectors:

- Password **shared** with someone (colleague/friend "just to check something")
- Password **reused** from a personal account breached elsewhere
- Phone given for **repair/handling** with apps and sessions still signed in
- Initial password delivered **insecurely** (TKT-003 principles violated)

### Containment Executed

```powershell
Import-Module ActiveDirectory

# 1. CONTAIN (contain before investigate)
Disable-ADAccount -Identity "ahassan"

# 2. Kill the credential (real password used at execution,
#    REDACTED in documentation — see Notes)
Set-ADAccountPassword -Identity "ahassan" -Reset `
    -NewPassword (ConvertTo-SecureString "<REDACTED>" -AsPlainText -Force)

# 3. Evidence for security team + manager
Get-ADUser -Identity "ahassan" -Properties Enabled, MemberOf, LastLogonDate,
    BadLogonCount, LastBadPasswordAttempt, PasswordLastSet |
    Select-Object Name, Enabled, MemberOf, LastLogonDate,
                  BadLogonCount, LastBadPasswordAttempt, PasswordLastSet
# Result: Enabled=False, fresh PasswordLastSet, BadLogonCount=0,
#         MemberOf={SG-Sales}
```

**MFA fix (Entra ID, per TKT-008):**
> Entra ID → Identity → Users → All users → Aisha →
> Authentication methods → **Require re-register multifactor
> authentication** — her compromised factor is wiped; she re-enrolls
> fresh. Plus (TKT-008 prevention): **Conditional Access → Named
> Locations** + **block legacy authentication** so a foreign 3 AM
> attempt can never silently reach her factor again.

**Escalation package:** timestamps + source country/IP, the APPROVED
factor (confirmed compromise), containment done, and the vector
hypotheses. Security team + manager informed — Tier 1 never handles a
confirmed compromise alone.

---

## Ticket C — The Group Rename (Priority 2)

**Found:** on Friday evening, during "cleanup," the group was renamed
`SG-Marketing` → `SG-Marketing-Old`. Monday morning, all four Marketing
users hit access denied.

**Why renaming breaks access — the SID lesson (TKT-004):**

The file server does not check the group's NAME — it checks the group's
**SID** embedded in each user's access token. Renaming does NOT change
the SID... so the permissions themselves survived. What broke is the
**name-to-SID resolution** wherever the name is referenced (share
permissions, scripts, documentation). The fix is to restore the name the
environment expects:

```powershell
# Identity = the group AS IT IS NOW; parameters = what it SHOULD be
Set-ADGroup -Identity "SG-Marketing-Old" `
    -SamAccountName "SG-Marketing" `
    -DisplayName "SG-Marketing"

# Verify
Get-ADGroup -Identity "SG-Marketing" |
    Select-Object Name, SamAccountName, GroupScope, GroupCategory
```

Users then need a **new logon session** to rebuild tokens with the
correctly-resolved group (TKT-004 lesson: token changes require new logon).

---

## Ticket A — Maria's Login (Priority 3)

Handled with the TKT-002 playbook:

- **Ask for the exact error, word for word** ("locked" vs "wrong
  password" vs "disabled" are three different diagnoses)
- **Forensics:** `Get-ADUser` → Enabled / LockedOut / PasswordExpired /
  BadLogonCount — evidence over the user's self-diagnosis
- Scope check: is anyone else in Sales affected? (In this queue: no —
  which itself is evidence it's her account, not the domain)

---

## Ticket B — The Formality With a Deadline (Priority 4)

**Corrected responses (important — two DIFFERENT people were confused here):**

> To Sofia, re: the **contractor:** *"Noted — the account will be built
> per our onboarding checklist (create with compliant password →
> SG-Sales membership verified against a reference user → force password
> change at first logon → create disabled, enable on their start day).
> I'll confirm completion by Thursday, well before they start."*

> To Sofia, re: **Robert's laptop** (TKT-007 loose end): *"Returned,
> OneDrive sync verified complete, and it's queued for reimage before
> reissue — browser-saved passwords and local caches survive user-account
> deletion."*

**What NOT to do because it's "no rush":** defer it into oblivion. The
cost of forgetting: the contractor arrives to no account → idle day one,
stressed manager, and a repeat of the TKT-004 pattern (incomplete
provisioning discovered weeks later). "No rush" means "not first," never
"not tracked."

---

## 🔍 The Hidden Lesson — A Skipped Checklist Item Came Back as an Incident

The exam's embedded finding: **Aisha's account was still ENABLED before
containment** — but TKT-006's checklist decision was *"create disabled,
enable Monday morning."* The disable correction was documented as a
pending action in TKT-006 **and never executed** (no verification
screenshot was ever produced).

Result: an enabled account with an unused password sat dormant for days
— and at 3 AM, someone used it. **Checklist items you skip don't
disappear; they come back as incidents.** The dormant-enabled account
was the attack surface. This is the single most valuable finding of the
entire project: process discipline isn't bureaucracy — it's security.

---

## Notes / Troubleshooting Artifacts

- ✅ **Positive security practice:** the password in the reset command
  was REDACTED before sharing the transcript/screenshots (shown as a
  dash string in documentation). Real, policy-compliant password was
  used at execution time. Sharing unredacted credentials in tickets,
  chats, or screenshots is a leak vector — redaction is correct handling.
- **Security hygiene evolution (honest record):** in TKT-006/007,
  credentials were shared in plaintext in transcripts; by TKT-010 the
  practice was corrected — passwords redacted before sharing. Lab
  environment only; affected accounts have since been reset/removed.
- Initial Ticket B response mistakenly merged Robert (departed, TKT-007)
  with the new contractor — caught and corrected during review. Two
  people, two tasks, never merged.

---

## Lessons Learned — Final Exam Summary

1. **Triage by risk, not by volume:** security event → team outage →
   single user → formality. Loud users do not set priorities; risk does.
2. **Approved factor ≠ blocked factor:** MFA blocking = suspected
   compromise; MFA approving while the user sleeps = CONFIRMED compromise.
   The response escalates accordingly.
3. **Contain, then investigate, then escalate** — and never handle a
   confirmed compromise solo.
4. **The file server checks SIDs, not names** — a rename breaks
   resolution, not permissions; restore the name, new logon for tokens.
5. **"No rush" still needs a tracked deadline** — forgotten formality
   tickets become day-one disasters.
6. **Skipped checklist items return as incidents** — the dormant enabled
   account was created by a shortcut in TKT-006 and exploited here.
7. **Redact credentials before anything leaves the console** — a habit
   that matured during this project (documented above).

---