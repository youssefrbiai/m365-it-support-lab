TKT-008-mfa-lockout-security-event.md

```markdown
# TKT-008 — MFA Lockout + Suspected Sign-In Compromise (David Chen)

**Date:** 27/09/2026
**Priority:** High
**SLA Target:** 30 minutes
**Status:** ✅ RESOLVED (contained + escalated to security team)
**Technician:** yousef
**Environment:** TAILWIND-DC1 (on-prem simulation of an M365/Entra ID scenario)

---

## Symptom

David Chen (Marketing) reported two problems in one ticket:

1. **MFA lockout:** got a new phone yesterday and traded in the old one
   at the store. This morning, Office sign-in asks him to approve an MFA
   notification — but it goes to the TRADED-IN phone. He cannot sign in
   from laptop, Outlook, or anything. Campaign deadline Thursday.
2. **🚨 Security event:** at 2 AM he received an email about a
   **sign-in attempt from another country.**

**Key lesson:** the MFA lockout is a routine ticket. The foreign sign-in
attempt makes this a **security event** — and the security event comes
FIRST.

---

## Triage Decision: Security First, Access Second

**Q: Restore access first or investigate the sign-in attempt first?**

→ **CONTAIN FIRST.** Disable the account and reset the password BEFORE
deep investigation. Reasoning: if the 2 AM attempt was an attacker, he
already has David's password and is only ONE factor away from entry.
You do not leave the door open while investigating slowly. If the
attacker gets in, he reaches company data — campaigns and deadlines do
not outweigh a compromise.

**Q: What else lives on the traded-in phone besides the Authenticator app?**

→ The old phone holds:
- The **Authenticator app** (his approval factor!)
- **Saved browser passwords** (what did David do with TKT-003's temp
  password? Possibly typed and saved it in the phone's browser)
- **Still-logged-in sessions** — Outlook/Teams/email apps
- **Cached corporate data** (files, message history)

Trading in a phone without wiping it = handing a stranger both factors.
The 2 AM "mystery visitor" may literally be whoever bought that phone.

---

## Execution — Containment Sequence (Lab Simulation)

The same containment pattern used in TKT-007's offboarding, applied to a
suspected compromise:

```powershell
Import-Module ActiveDirectory

# 1. Block sign-in immediately (containment before investigation)
Disable-ADAccount -Identity "dchen"
Get-ADUser -Identity "dchen" -Properties Enabled | Select-Object Name, Enabled
# Result: David Chen — Enabled: False ✅

# 2. Force password reset (kill any stolen password's usefulness)
Set-ADAccountPassword -Identity "dchen" -Reset `
    -NewPassword (ConvertTo-SecureString "<RESET-PASSWORD>" -AsPlainText -Force)

# 3. Review the attacker's potential reach — group access
Get-ADUser -Identity "dchen" -Properties MemberOf |
    Select-Object -ExpandProperty MemberOf
# Result: CN=SG-Marketing,... (Marketing resources only)

# 4. Collect account-level evidence for the security team
Get-ADUser -Identity "dchen" -Properties LastLogonDate, BadLogonCount,
    LastBadPasswordAttempt |
    Select-Object Name, LastLogonDate, BadLogonCount, LastBadPasswordAttempt
# Result: LastLogonDate = (blank), BadLogonCount = 0
```

**Evidence reading (say it out loud in the ticket):** blank LastLogonDate
+ BadLogonCount 0 = **no confirmed compromise on this system** — evidence,
not assumption. This is exactly the baseline a security team needs.

### Why the same sequence serves two different purposes

TKT-007 (offboarding) and TKT-008 (containment) use the IDENTICAL
commands — disable + reset. The difference is the **undo path**:

- Offboarding: never comes back. Permanent by design.
- Containment: a temporary holding pattern — lifted as soon as the user
  is verified safe and MFA methods are re-registered.

Same technique, opposite lifecycle. The commands don't define the
operation; the business context does.

---

## The MFA Fix (Entra ID Flow)

**Admin path to reset a user's MFA when the device is lost:**

> Entra ID → Identity → Users → All users → select user →
> **Authentication methods** (under Manage) →
> **Require re-register multifactor authentication**

**What it does:** clears David's existing MFA methods (the ones living on
the traded-in phone). At next sign-in he is forced to register **fresh**
methods from scratch — exactly what you want when the old methods are in
a stranger's pocket.

**Alternative methods so this never happens again:**

1. **TAP (Temporary Access Pass)** — admin-issued, time-limited code that
   lets David sign in and enroll new MFA methods without any old device
2. **FIDO2 security key / hardware token** — the device-INDEPENDENT
   option: it doesn't live on any phone, so a lost/traded phone can never
   take it down. (Phone call/SMS to an office number is a weaker
   device-independent fallback.)

---

## Security Escalation (Tier 1 → Security Team)

**Decision: ESCALATE — do not handle a suspected compromise alone.**

**Handover package for the security team:**

1. **Timestamp + source country/IP** of the 2 AM sign-in attempt
2. **Whether it succeeded or failed** (MFA blocked it — but see trap below)
3. **Containment already performed:** account disabled, password reset,
   MFA re-registration required
4. **The traded-in-phone timeline** — old device left David's possession
   yesterday with Authenticator + possibly saved credentials on it

**The trap question — "it FAILED, so nothing bad happened, no report needed":**

→ **FALSE. A failed attempt must still be reported.**
- A failed MFA attempt tells the attacker his **password is CORRECT** —
  MFA is the only remaining barrier, and he needs just one more factor
- A 2 AM attempt is MORE suspicious than a 2 PM one: outside duty hours
  means no legitimate user, and nobody is watching the account
- Failed ≠ harmless. Reported ≠ overreacting.

---

## Prevention

**David's pre-trade-in checklist (for the future):**

1. **Transfer Authenticator first** — cloud backup/restore or re-enroll
   with admin help BEFORE giving up the device
2. **Sign out of all work apps** and remove saved passwords from the
   phone's browser
3. **Full factory wipe** of the phone before handover

**Conditional Access rule that would have neutralized the 2 AM attempt
without bothering David at all:**

> **Conditional Access → Named Locations** — allow sign-ins only from
> countries where the company operates; block or challenge everything
> else. A foreign 2 AM attempt is then blocked automatically at the
> door — David never even sees the notification.
>
> (Related hardening: **block legacy authentication** — legacy mail
> protocols skip MFA entirely, which is exactly the hole attackers
> hunt for.)

---

## Lessons Learned

1. **Security events outrank convenience tickets** — contain first,
   investigate second, restore access last.
2. **A lost MFA device is a credential event, not just a lockout** — the
   factor walked out the door with the phone.
3. **Failed sign-in attempts get reported, always** — they confirm the
   attacker's password works and reveal what he's trying.
4. **Require re-register MFA + TAP** is the standard recovery pair for
   lost-device lockouts.
5. **Named Locations (Conditional Access)** stops foreign attempts
   automatically — prevention beats reaction.
6. **Same commands, different purpose:** disable + reset serves
   offboarding (permanent) and containment (temporary) — context defines
   the operation.

---
