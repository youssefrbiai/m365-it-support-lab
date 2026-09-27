TKT-006-new-hire-onboarding.md
```markdown
# TKT-006 — New Hire Onboarding (Aisha Hassan)

**Date:** 27/09/2026
**Priority:** Medium (deadline: Friday, start date Monday)
**SLA Target:** Complete checklist before start date
**Status:** ✅ RESOLVED (one scheduled action for Monday — see below)
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## Request

HR (Sofia Ramos) submitted an onboarding request:

> "Aisha Hassan joins the Sales team on Monday. She needs: a domain
> account, access to the same Sales resources as the rest of the team,
> and her account ready BEFORE she walks in — first-day impression
> of IT matters."

---

## Onboarding Checklist (Ordered)

1. Create account with correct attributes (name, department, title, UPN)
2. Set compliant initial password (policy: 14+ chars, complexity)
3. Add to SG-Sales (same access as existing Sales team)
4. Force password change at first logon
5. Verify everything against a reference user (evidence, not memory)
6. Account state decision: create disabled, enable Monday morning
7. Document + notify HR/manager that onboarding is complete

---

## Execution

```powershell
Import-Module ActiveDirectory

# 1. Create the account
New-ADUser -Name "Aisha Hassan" `
    -GivenName "Aisha" `
    -Surname "Hassan" `
    -SamAccountName "ahassan" `
    -UserPrincipalName "ahassan@tailwindtraders.internal" `
    -Department "Sales" `
    -Title "Sales Representative" `
    -AccountPassword (ConvertTo-SecureString "QECXG1@\$13eIgsch&*8dAQ~" -AsPlainText -Force) `
    -Enabled $true

# 2. Group membership — same as Sales team
Add-ADGroupMember -Identity "SG-Sales" -Members "ahassan"

# 3. Force password change at first logon (TKT-003 lesson)
Set-ADUser -Identity "ahassan" -ChangePasswordAtLogon $true

# 4. Full verification — one command
Get-ADUser -Identity "ahassan" -Properties * |
    Select-Object Name, Enabled, Department, Title,
                  PasswordExpired, PasswordLastSet, MemberOf
```

**Password compliance check:** `QECXG1@$13eIgsch&*8dAQ~` =
**23 characters**, upper + lower + digits + multiple symbols —
exceeds MinPasswordLength (14) and complexity requirement. Pattern
completely different from previous lab passwords.

---

## Verification Against Reference User (Key Technique)

Rather than trusting memory or the HR form, compared Aisha's group
access against an existing Sales team member (Maria, mgonzalez):

```powershell
$New = (Get-ADUser ahassan -Properties MemberOf).MemberOf
$Ref = (Get-ADUser mgonzalez -Properties MemberOf).MemberOf

Compare-Object -ReferenceObject $Ref -DifferenceObject $New |
    Where-Object SideIndicator -eq "<="
# Groups Maria has that Aisha is MISSING: (empty)
```

**Result:** Aisha has SG-Sales; the "missing" comparison returned
**empty** — nothing missing.

**Why reference-user comparison is safer:** it verifies against
*evidence, not memory or the request form*. TKT-004 proved that request
forms get half-completed (HR's transfer ticket was never finished) and
that nobody noticed for a week. Compare-Object against a known-good
team member catches that class of error instantly.

---

## Account State Decision (Security Reasoning)

**Decision: account should be DISABLED until Monday morning.**

Reasoning: an enabled account sitting unused for 3 days is
**attack surface** — nobody is watching it, the initial password sits
in it, and compromised dormant accounts are a classic attacker entry
point. Enable it when she actually walks in, then she changes the
password at first logon (PasswordExpired = True already set).

**Password handling rules:**
- The initial password is known ONLY to Aisha — delivered securely at
  first logon, never to her manager, never by email
- Nobody touches or changes anything on the account between creation
  and start date

 **troubleshooting artifact:** the account was
initially created with `-Enabled $true` before the checklist review
raised the disabled-until-Monday decision. The original screenshot
shows Enabled = True. **Correction action:** the account must be
disabled before the weekend — command below — and re-enabled Monday.

```powershell
# CORRECTION (to run before Monday):
Disable-ADAccount -Identity "ahassan"
Get-ADUser -Identity "ahassan" -Properties Enabled | Select-Object Name, Enabled
# Expected: Enabled = False

# MONDAY MORNING (start of shift):
Enable-ADAccount -Identity "ahassan"
```

---

## Verification Summary

- ✅ Account created with correct attributes (Sales / Sales Representative)
- ✅ SG-Sales membership confirmed, reference-user comparison = no gaps
- ✅ PasswordExpired = True (forced change at first logon)
- ✅ Password exceeds domain policy (23 chars, all character classes)
- ⚠️ Account state: correction to disabled pending before Monday

---

## Key Concept: M365 / Entra ID Mapping

On-prem steps vs the same onboarding in a Microsoft 365 tenant:

| On-prem (this ticket) | M365 equivalent |
|---|---|
| New-ADUser | Create user in Admin Center / New-MgUser |
| SG-Sales security group | **Security group** in Entra ID → grants permissions (e.g., SharePoint document library access) |
| — | **Microsoft 365 Group** → a whole workspace: its own SharePoint site + shared mailbox + Teams team, all granted by one membership |
| Force change at first logon | User prompted to set password at first sign-in (+ MFA registration) |
| — | **License assignment:** a new M365 user has NO mailbox, NO Teams, NO Office apps until a license (e.g., Microsoft 365 Business Premium) is assigned to their account. Account first, license can follow — but nothing works until licensed |

Security group = permission grants. M365 Group = workspace bundle.
That distinction is a standard interview question.

---

## Lessons Learned

1. **Onboarding is checklist-driven** — unordered good intentions miss steps;
   the checklist's verification items (5–7) are what separate "I did it"
   from "I proved it's done."
2. **Verify against a reference user, not against the request form** —
   forms get half-completed (TKT-004).
3. **Dormant enabled accounts are attack surface** — create disabled,
   enable on start day, force password change at first logon.
4. **Initial password distribution is a security event:** one person,
   one secure channel, expired at first use (TKT-003 principles).
5. **Write the ticket to match reality, including mistakes** — the
   enabled-vs-disabled mismatch is documented, not hidden.

---