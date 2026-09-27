TKT-003-password-reset-identity-verification.md

```markdown
# TKT-003 — Password Reset with Identity Verification (David Chen)

**Date:** 25/09/2026
**Priority:** Medium
**SLA Target:** 15 minutes
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## Symptom

User David Chen (Marketing) reported he forgot his password — and forgot
which password he last used. He was calling from a **personal phone**
(laptop at the office) and made three requests:

1. Reset the password and **tell him the new password verbally** over the phone
2. Reset without identity verification ("you know me")
3. Have the technician **stay logged in as him** on his office laptop so he
   wouldn't have to type the new password

---

## 🚨 Security Gate (Before Any Technical Work)

> A password reset given to the wrong person = handing an attacker the keys.
> Every one of David's requests matches known **social engineering**
> patterns, even though the caller was (probably) legitimate.

### Identity Verification Performed (Two-Factor, Out-of-Band)

1. **Callback to number on file** — hung up and called David back on the
   office phone number recorded in AD/HR (verified via
   `Get-ADUser dchen -Properties OfficePhone, Manager, Department`),
   NOT the number he called from.
2. **Manager confirmation out-of-band** — his manager confirmed through a
   known internal channel that David was genuinely requesting a reset.

*(Security questions were considered and rejected: answers can be
researched on social media/LinkedIn — weak verification.)*

### Requests Refused

| Request | Why Refused |
|---|---|
| Verbal password over the phone | Unsecure channel; no way to verify listener; password would live in call history/memory |
| "Stay logged in as me" | Destroys **accountability / non-repudiation** — every action on the account must be attributable to its owner. If the caller were an attacker, we'd be handing them an authenticated session with zero trace of who really acted |

---

## Diagnosis

- User exists: `dchen` (David Chen, Marketing), account Enabled.
- No account issue — this is a **credential loss** incident, not an
  access/lockout problem (contrast with TKT-002).
- Risk: the "lost" password may have been compromised (which is often
  WHY users lose track of it) — reset must be treated as a security
  event, not a convenience.

---

## Fix

```powershell
# Reset with a new temporary password
Set-ADAccountPassword -Identity "dchen" -Reset `
    -NewPassword (ConvertTo-SecureString "<TEMP-PASSWORD>" -AsPlainText -Force)

# Force the user to set his own password at first logon
Set-ADUser -Identity "dchen" -ChangePasswordAtLogon $true

# Verify
Get-ADUser -Identity "dchen" -Properties PasswordExpired, PasswordLastSet |
    Select-Object Name, PasswordExpired, PasswordLastSet
# Result: PasswordExpired = True ✅
```

### Why Force Change-at-Logon?

The temp password traveled through **unsecure channels** (spoken on a
call, typed in a ticket). Anyone who overheard or read it could log in.
Forcing a change at first logon means:

- The temp password is valid **once**, for **one person**, for **minutes**
- The exposure window is tiny
- AD enforces the change itself (`PasswordExpired: True`) — we don't
  have to trust the user to "get around to it"

### Temp Password Delivery Procedure

- Temp password given only AFTER verification passed
- User required to change it immediately at first logon
- Password never written into the ticket body or sent by email/chat

---

## Verification

- ✅ `PasswordExpired: True` — AD will demand a new password at next logon
- ✅ Account enabled, group memberships intact
- ✅ User informed: change password at next logon, do not reuse old passwords
  (domain enforces history of 24)

---

## Key Concept: M365 Mapping

In a Microsoft 365 tenant, most password-reset tickets never reach the
helpdesk because of **SSPR — Self-Service Password Reset**:

- User visits the "Forgot my password" link on the sign-in page
  (passwordreset.microsoftonline.com)
- Verifies identity with MFA methods (authenticator app, phone, email)
- Resets their own password — no admin, no ticket

This is why password-reset ticket volume drops dramatically in M365
shops compared to on-prem AD. SSPR is a standard interview topic for
M365 support roles.

---

## Notes / Dead Ends / Improvements

- ⚠️ **Improvement noted:** during the lab, the temp password was typed in
  plaintext into the console, so it remains visible in PowerShell command
  history — a real-world leak vector. Better practice next time: generate
  the password into a variable at runtime, or use a random-password
  generator, so it never sits in console history or transcripts.
- Security questions rejected as a verification method (researchable answers).
- User's own framing ("locked") would have sent the ticket the wrong way;
  treating it as credential loss + security event was the correct frame.

---

## Lessons Learned

1. **Identity verification precedes any password operation.** Two-factor,
   out-of-band (callback to number on file + manager confirmation).
2. **Never share passwords verbally or through ticket/email/chat.**
   Temp password + forced change-at-logon = minimal exposure window.
3. **Refuse session-sharing requests** — non-repudiation and
   accountability must be preserved, even for friendly legitimate users.
4. **SSPR** is the M365 answer to this entire ticket category.

---

*Methodology: Identify → Isolate → Test → Root Cause → Fix → Verify → Document*
*Related: TKT-001 (user lifecycle / DC discovery), TKT-002 (locked vs disabled — access investigation)*
```