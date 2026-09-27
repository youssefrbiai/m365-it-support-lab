TKT-001-user-lifecycle-ad-to-m365-mapping

```markdown
# TKT-001: User Lifecycle Management — AD to M365 Mapping

| Field | Value |
|-------|-------|
| **Ticket ID** | TKT-001 |
| **Date** | 25/09/2026 |
| **Environment** | TAILWIND-DC1 (Domain Controller, tailwindtraders.internal) |
| **Status** | ✅ Resolved |
| **Priority** | Low (training exercise) |
| **Technician** | Yousef Rbiai |

---

## 1. IDENTIFY

**Task:** Simulate the complete user lifecycle in Active Directory and map each operation to its Microsoft 365 Admin Center equivalent.

**Business Context:** User onboarding/offboarding is a daily IT Support task. Understanding both on-prem AD and cloud M365 proves hybrid environment capability — critical for UAE employers running mixed infrastructure.

**Scope:** Create user → assign to group → reset password → disable/enable → delete → verify each step.

---

## 2. ISOLATE

**Environment Discovery:**
```powershell
# Verify machine role
(Get-CimInstance Win32_ComputerSystem).DomainRole
```
**Result:** DomainRole = 4 or 5 (Domain Controller)

**Critical Finding:** Domain Controllers do not have a local SAM database. All `*-LocalUser` cmdlets fail because there are no local accounts on a DC.

**Initial Error Encountered:**
```
New-LocalUser : Unable to update the password. The value provided for the new password 
does not meet the length, complexity, or history requirements of the domain.
```

**Secondary Errors (Cascade):**
```
Add-LocalGroupMember : Principal jsmith was not found.
Get-LocalUser : User jsmith was not found.
Set-LocalUser : User jsmith was not found.
```

---

## 3. TEST

**Password Policy Verification:**
```powershell
Get-ADDefaultDomainPasswordPolicy
```

**Results:**
| Setting | Value |
|---------|-------|
| ComplexityEnabled | True |
| MinPasswordLength | 14 |
| PasswordHistoryCount | 24 |
| MaxPasswordAge | 42 days |
| LockoutThreshold | 0 |

**Troubleshooting Artifact:**
Typo encountered: `get-addefaultdomainpasswordplicy` → `CommandNotFoundException`
Corrected to: `Get-ADDefaultDomainPasswordPolicy`

---

## 4. ROOT CAUSE

The original script assumed a **local workstation** environment. A **Domain Controller** stores all identities in Active Directory — there is no local user database.

**Lesson:** Error messages describe symptoms, not root causes. The "password complexity" error was misleading; the real issue was using local cmdlets on a domain-joined DC.

---

## 5. FIX

**Corrected Approach — ActiveDirectory Module:**

| Step | AD Cmdlet | M365 Admin Center Equivalent |
|------|-----------|------------------------------|
| Create user | `New-ADUser -Name "jsmith" -GivenName "John" -Surname "Smith" -SamAccountName "jsmith" -UserPrincipalName "jsmith@tailwindtraders.internal" -Path "OU=Sales,DC=tailwindtraders,DC=internal" -AccountPassword (Read-Host -AsSecureString) -Enabled $true` | Users → Active users → Add a user |
| Create group | `New-ADGroup -Name "SG-Sales" -GroupScope Global -GroupCategory Security` | Groups → Security group → Create |
| Add to group | `Add-ADGroupMember -Identity "SG-Sales" -Members "jsmith"` | Groups → SG-Sales → Members → Add |
| Verify user | `Get-ADUser -Identity "jsmith" -Properties Enabled, LastLogonDate` | Users → Active users → select user |
| Reset password | `Set-ADAccountPassword -Identity "jsmith" -Reset -NewPassword (Read-Host -AsSecureString)` | Users → jsmith → Reset password |
| Block sign-in | `Disable-ADAccount -Identity "jsmith"` | Users → jsmith → Block sign-in |
| Unblock | `Enable-ADAccount -Identity "jsmith"` | Users → jsmith → Unblock sign-in |
| Delete user | `Remove-ADUser -Identity "jsmith"` | Users → jsmith → Delete user |

**Note:** In the screenshots, the `Add-LocalGroupMember` error "Group Users was not found" confirms the DC limitation — local groups do not exist on domain controllers.

---

## 6. VERIFY

**Verification Commands Executed:**

```powershell
# User exists and is enabled
Get-ADUser -Identity "jsmith" | Select-Object Name, Enabled, SamAccountName

# Group membership confirmed
Get-ADGroupMember -Identity "SG-Sales"

# Account disabled state verified
Get-ADUser -Identity "jsmith" -Properties Enabled | Select-Object Enabled
# Expected: False

# Account re-enabled
Get-ADUser -Identity "jsmith" -Properties Enabled | Select-Object Enabled
# Expected: True

# Deletion confirmed (expected error)
Get-ADUser -Identity "jsmith"
# Expected: "Cannot find an object with identity: 'jsmith'"
```

**Screenshot Evidence:** See `05_Screenshots/TKT-001/` — PowerShell output showing successful creation, group operations, and deletion verification.

---

## 7. DOCUMENT

### Key Lessons Learned

1. **Environment First:** Always verify machine role (DC vs workstation) before scripting. `(Get-CimInstance Win32_ComputerSystem).DomainRole` takes 5 seconds and prevents 30 minutes of confusion.

2. **Error Messages Lie:** The "password complexity" error was a red herring. The real issue was cmdlet family mismatch.

3. **Conceptual Parity:** AD and M365 user lifecycles are identical in concept — only the interface differs. `Disable-ADAccount` = "Block sign-in" in M365.

4. **Cascade Failures:** One root cause (wrong cmdlet family) created 4+ downstream errors. Fix the root, not the symptoms.

### Skills Demonstrated

- Active Directory user management (create, modify, delete)
- PowerShell troubleshooting and error interpretation
- Environment assessment and adaptation
- On-prem to cloud concept mapping
- Documentation discipline (ticket methodology)

### M365 Concepts Practiced (Theoretical — No Tenant Access)

| Concept | AD Equivalent | M365 Equivalent |
|---------|---------------|-----------------|
| User provisioning | New-ADUser | Users → Add user |
| Group-based management | Security Groups | Microsoft 365 Groups / Security Groups |
| Access control | Disable-ADAccount | Block sign-in |
| Password policy | Default Domain Policy | Entra ID Password Protection |
| Soft delete | AD Recycle Bin | Deleted users (30-day retention) |

### Block Sign-in vs Delete User — Decision Matrix

| Scenario | Action | Reason |
|----------|--------|--------|
| Employee on leave | Block sign-in | Preserves data, mailbox, licenses; reversible |
| Suspected compromise | Block sign-in | Immediate containment; investigate later |
| Employee terminated | Delete user | Removes access; mailbox converts to shared after 30 days |
| Contractor finished | Delete user | Clean up guest/external accounts |

**Best Practice:** Block first, delete later. You can always delete a blocked account; you cannot always recover a deleted one.

---

**Ticket Closed:** 25/09/2026  
```