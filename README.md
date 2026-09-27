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
- AD user lifecycle: create → groups → reset → block → delete
  (`New-ADUser`, `Set-ADAccountPassword`, `Enable/Disable-ADAccount`, group management)
- Account forensics: `Get-ADUser -Properties *` (Enabled, LockedOut,
  BadLogonCount, LastLogonDate)
- Security containment: disable + password reset under suspected/confirmed compromise
- Service management: Print Spooler root-cause cleaning (stop → clear → start)
- Reference-user comparison: `Compare-Object` against known-good access patterns

**M365/Entra knowledge (mapped from hands-on AD work):**
- Admin Center user/license flows, SSPR, MFA re-registration, TAP
- Conditional Access (Named Locations, legacy auth blocking)
- Shared mailbox conversion, license data-retention rules, group-based licensing
- On-prem ↔ cloud mapping documented per ticket

**Professional discipline:**
- Full ticket documentation with honest dead-ends and artifacts
- De-escalation and user communication scripts
- Escalation judgment (when to hand to security/finance/HR — and what to hand them)
- Credential redaction in all shared documentation

## Key Incidents Worth Reading
- **TKT-002:** why `BadLogonCount = 0` disproved the user's story
- **TKT-006 → TKT-010:** how one skipped checklist step (enabled dormant account)
  returned as a 3 AM confirmed compromise
- **TKT-007:** the disable + reset defense-in-depth pattern

---

## Repository Structure

M365-IT-Support-Lab/
├── README.md                          ← you are here
├── 00_Context/
│   ├── PROJECT-STATE.md               ← running project record
│   ├── ENVIRONMENT.md                 ← lab environment documentation (v1.1)
│   └── RETROSPECTIVE.md               ← growth, mistakes, interview story
├── 04_Incident_Tickets/
│   ├── TKT-001-user-lifecycle-script-failure.md
│   ├── TKT-002-account-access-investigation.md
│   ├── TKT-003-password-reset-identity-verification.md
│   ├── TKT-004-group-access-intermittent.md
│   ├── TKT-005-printer-deescalation.md
│   ├── TKT-006-new-hire-onboarding.md
│   ├── TKT-007-offboarding-departure.md
│   ├── TKT-008-mfa-lockout-security-event.md
│   ├── TKT-009-license-revocation-access-loss.md
│   └── TKT-010-monday-storm-final-exam.md

## Roadmap
- [x] Project Charter
- [x] VM / Domain Controller verification
- [x] Microsoft 365 fundamentals (Microsoft Learn)
- [x] 10-incident ticket series with full documentation
- [x] Retrospective + portfolio packaging
- [ ] M365 trial tenant (Developer Program) — convert knowledge-based tickets to hands-on
- [ ] MS-900: Microsoft 365 Fundamentals certification
- [ ] MD-102 (Endpoint Administrator) or SC-900 (Security)

---

*Every ticket documents real execution or honestly-labeled knowledge work.
Mistakes are included on purpose — they are the evidence of learning.*
