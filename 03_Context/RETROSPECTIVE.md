RETROSPECTIVE.md

# Project Retrospective — M365 IT Support Administration Lab

**Author:** yousef
**Date:** 28/09/2026
**Scope:** TKT-001 through TKT-010 (complete series)
**Purpose:** Honest self-assessment of growth, mistakes, and lessons —
written to be reused as interview answers, because every word of it is true.

---

## 1. Growth — What I Can Do Now That I Couldn't Before

Three concrete capabilities, built ticket by ticket:

### 1. Respond to an active attack
When a sign-in compromise happens, I don't freeze — I contain immediately:
disable the account, reset the password, revoke the compromised MFA
methods (Require re-register multifactor authentication), and hand a
structured evidence package to the security team. **Contain first,
investigate second.** (TKT-008, TKT-010)

### 2. Diagnose from evidence, not assumptions
I learned to read account forensics and let the system tell the story.
Example: seeing `Enabled: False` with `BadLogonCount: 0` and knowing the
password was never even checked — a disabled account rejects before
password validation — instead of trusting the user's story or my first
guess. "I'm locked out" and "my account is disabled" feel identical to
users; the evidence separates them. (TKT-002, TKT-004)

### 3. Triage under pressure by risk, not noise
On the final day, four tickets arrived in three minutes — an angry user,
a whole-team outage, a security event, and a "no rush" formality. I
ordered them correctly: **security event → team outage → single user →
formality.** Loud does not mean urgent. A shouting user sets volume, not
priority. (TKT-010)

---

## 2. My Most Instructive Mistake — The Full Arc

### The small version: the printer ticket (TKT-005)
The user was angry and I was reacting to her urgency. I initially assumed
the problem was the internet, the cable, or the computer. The real issue
was a corrupted job stuck in the print spooler cache — and the reason it
kept coming back was that previous fixes cleared the queue but never
removed the corrupt file itself.

Two lessons from that one ticket:
- **Check scope before guessing** — only 1 of 8 users was affected, which
  already pointed away from the printer itself.
- **A fix that doesn't remove the cause is a symptom fix** — and symptom
  fixes create repeat tickets.

### The serious version: the skipped checklist step (TKT-006 → TKT-010)
The same class of mistake returned at a much higher stakes level. During
onboarding (TKT-006), I documented a security decision — *create the new
hire's account disabled until her start date* — and then never actually
ran the command. No verification screenshot was ever produced, and nobody
(not even me) went back to check.

That dormant enabled account sat for days with an unused password — and
in the final ticket (TKT-010), it came back as a **3 AM confirmed
compromise**: a foreign sign-in attempt where her MFA factor was approved
while she slept.

### The lesson that hardened in me
**Skipped checklist items don't disappear — they return as incidents.**
Process discipline isn't bureaucracy; it IS security. Now, nothing on a
checklist is "done" for me until it is verified with evidence — because
in my own project, I watched a shortcut become an incident.

### A second, quieter improvement: credential hygiene
In TKT-006 and TKT-007, I shared credentials in plaintext in transcripts
and documentation. By TKT-010, I had corrected the habit — passwords were
redacted before anything left the console (lab environment only; affected
accounts have since been reset/removed). The improvement is documented,
the earlier exposure is honestly noted. Growth includes fixing how you
document, not just how you fix.

---

## 3. The 60-Second Interview Story (TKT-006 → TKT-010)

> "On my final day, I contained a confirmed compromise on a new
> employee's account — a 3 AM foreign sign-in where the MFA factor was
> approved while she slept. When I investigated how her password leaked,
> I found something uncomfortable: the account was still enabled days
> after creation, because in the onboarding ticket I had documented the
> decision to 'create it disabled until her start date' — and then never
> actually ran the command. I skipped one checklist verification step.
>
> That skipped step created a dormant enabled account — the exact attack
> surface that got used. I fixed the compromise with containment:
> disable, password reset, MFA re-registration, escalation to security.
>
> But the real lesson wasn't the fix — it's that process discipline IS
> security. Now, when I complete a checklist, I don't consider anything
> done until it's verified with evidence — because in my own project, I
> watched a shortcut become an incident."

*Why this story works: it is true, self-built, cause-and-effect complete,
and it turns my worst moment into proof that I convert mistakes into
permanent habits.*

---

## 4. Ticket-by-Ticket — What Each One Taught Me

| # | Ticket | The lesson in one line |
|---|--------|------------------------|
| 001 | User lifecycle script failure | Error messages lie — verify the environment first (the VM was a DC; local account cmdlets were the wrong tools entirely) |
| 002 | Account access investigation | Trust the system's evidence, not the user's self-diagnosis — "locked" vs "disabled" are different problems with different fixes |
| 003 | Password reset + identity verification | Identity verification comes BEFORE any password operation — a reset given to the wrong person is an account takeover |
| 004 | Intermittent group access | Windows builds access tokens at logon — the fix isn't done until the session is new; and an empty result is evidence too |
| 005 | Printer + angry user | Symptom fixes create repeat tickets; and a technically perfect fix delivered badly still loses the customer |
| 006 | New hire onboarding | Onboarding is checklist-driven — and verify against a reference user, not against memory or the request form |
| 007 | Employee offboarding | Defense in depth (disable + password reset); data decisions belong to HR/legal, never IT; the device outlives the user account |
| 008 | MFA lockout + security event | A password alone is not safety at all — MFA is what blocks the attacker; and a FAILED attempt still gets reported, because it confirms the attacker's password works |
| 009 | License revocation | Licenses are money: reclaiming/assigning is a financial decision with IT evidence; before ANY license removal, secure the data first (shared mailbox + OneDrive access) — and always inform/coordinate with IT so active users are never harvested |
| 010 | Monday Morning Storm | Everything at once: triage by risk (security → team → user → formality), contain confirmed compromises, and never merge two people's tasks into one response |

---

## 5. Where I Am Now — Honest Position

**Strong (hands-on proven):**
- Active Directory user lifecycle, forensics, containment — executed for real on a Domain Controller
- Incident documentation with the Identify → Isolate → Test → Root Cause → Fix → Verify → Document methodology
- Communication: de-escalation scripts, callback discipline, escalation judgment
- Triage under pressure

**Known gap (documented, not hidden):**
- No Microsoft 365 tenant hands-on — Admin Center/Entra flows are knowledge-based
  (mapped from my AD work), not executed. Next step: M365 Developer Program /
  trial tenant to convert tickets 006–009 into hands-on as well.

**Certification roadmap:**
MS-900 (Microsoft 365 Fundamentals) → MD-102 (Endpoint Administrator) or SC-900 (Security)

---