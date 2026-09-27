
# M365 IT Support Administration Lab — yousef

> A 10-ticket, hands-on IT support project simulating a full Tier 1 helpdesk
> experience: from user lifecycle basics to security incident response —
> executed on a real Windows Server Domain Controller, documented like
> production tickets.

---

## The Environment (Built, Not Borrowed)

- **Windows Server Domain Controller** (`TAILWIND-DC1`, tailwindtraders.internal)
  running in Oracle VirtualBox on macOS
- Real **Active Directory** administration via PowerShell (ActiveDirectory module)
- Full incident documentation using the methodology:
  **Identify → Isolate → Test → Root Cause → Fix → Verify → Document**

## The Ticket Series

| # | Ticket | Core Skills |
|---|--------|-------------|
| 001 | User lifecycle script failure | Root-cause analysis: error messages lie — verified environment first (DC vs local SAM) |
| 002 | Account access investigation | Locked vs disabled, forensic evidence over user self-diagnosis |
| 003 | Password reset | Identity verification, social engineering defense, non-repudiation |
| 004 | Intermittent group access | Kerberos tokens, group SIDs, "the fix isn't done until the session is new" |
| 005 | Recurring printer + angry user | Root vs symptom fixes, de-escalation scripts, repeat-ticket prevention |
| 006 | New hire onboarding | Checklist discipline, reference-user verification, dormant-account security |
| 007 | Employee offboarding | Defense in depth (disable + reset), shared mailboxes, Intune remote wipe, HR/legal data rules |
| 008 | MFA lockout + security event | Contain-first response, Require re-register MFA, TAP, Conditional Access |
| 009 | License revocation | License↔service mapping, 30-day data rules, group-based licensing, budget escalation |
| 010 | Monday Morning Storm (FINAL) | Multi-incident triage by risk, confirmed vs suspected compromise, SID vs name resolution |

## Key Competencies Demonstrated

**Hands-on (executed on the DC):**
- AD user lifecycle: create → groups → reset → block → delete (`New-ADUser`, `Set-ADAccountPassword`, `Enable/Disable-ADAccount`, group management)
- Account forensics: `Get-ADUser -Properties *` (Enabled, LockedOut, BadLogonCount, LastLogonDate)
- Security containment: disable + password reset under suspected/confirmed compromise
- Service management: Print Spooler root-cause cleaning (stop → clear → start)
- Reference-user comparison: `Compare-Object` against known-good access patterns

**M365/Entra knowledge (mapped from hands-on AD work):**
- Admin Center user/license flows, SSPR, MFA re-registration, TAP
- Conditional Access (Named Locations, legacy auth blocking)
- Shared mailbox conversion, license data-retention rules, group-based licensing
- On-prem ↔ cloud mapping for every operation (documented per ticket)

**Professional discipline:**
- Full ticket documentation with honest dead-ends and artifacts
- De-escalation and user communication scripts
- Escalation judgment (when to hand to security/finance/HR — and what to hand them)
- Credential redaction in all shared documentation

## Key Incidents Worth Reading
- **TKT-002:** why `BadLogonCount = 0` disproved the user's story
- **TKT-006 → TKT-010:** how one skipped checklist step (enabled dormant account) returned as a 3 AM confirmed compromise
- **TKT-007:** the disable + reset defense-in-depth pattern
```

---

## 📦 Deliverable 2 — Project Retrospective

Answer these in `00_Context/RETROSPECTIVE.md` — **in your own words**, because you will literally reuse these answers in interviews:

1. **The growth question:** What could you do after TKT-010 that you couldn't do before TKT-001? Name at least 3 concrete things (not "learned PowerShell" — be specific: *"I can look at `Enabled: False, BadLogonCount: 0` and know the password was never checked"*).
2. **The mistake question:** What was your most instructive mistake in this project? (You have a rich menu: the typo, the skipped disable step, the plaintext passwords, the Robert/contractor mix-up. Pick one and tell its story properly — interviewers ask this to see if you *learn*, not if you're perfect.)
3. **The interview question:** In 60 seconds, tell the story of TKT-006 → TKT-010 as one narrative. That's your strongest interview asset — a self-built cause-and-effect arc nobody could fabricate.

---

## 🎓 My Final Assessment — Honest

**What you built:** a genuinely respectable foundation. The triage instinct (risk over noise), the evidence-over-stories habit, the containment reflex, and the communication scripts — those are the things that make a Tier 1 tech promotable. Your ticket folder is a real portfolio.

**What's still true (from your own Environment Doc):** no M365 tenant hands-on. Your AD work is the closest free analog, but interviewers for M365 roles will ask Admin Center specifics.

**Recommended next steps, in order:**
1. **Retry the M365 Developer Program** (or a trial tenant) — free sandbox = your tickets 006–009 become hands-on too
2. **MS-900** (Microsoft 365 Fundamentals) cert — cheap, achievable now, signals commitment
3. Then **MD-102** (Endpoint Administrator) or SC-900 (Security) depending on direction
4. Keep the streak habit: one Learn module or lab touch per day

---
