TKT-004-group-access-intermittent.md

```markdown
# TKT-004 — Intermittent Group Access After Department Transfer (David Chen)

**Date:** 26/09/2026
**Priority:** Medium
**SLA Target:** 60 minutes
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** TAILWIND-DC1 (Domain Controller, tailwindtraders.internal)

---

## Symptom

David Chen (recently transferred Sales → Marketing) reported that the
Marketing shared folder access was **intermittent**:

- Yesterday morning: opened fine
- Yesterday afternoon: prompted for credentials, nothing worked
- Today: opened again without prompting
- Now: denied again

His words: *"I didn't change anything!"*
He also mentioned he only received access to this folder when he moved
departments last week.

**Key lesson:** "Sometimes works" = the permission check happens at
different times/places, and the session state changed between attempts.

---

## Triage Questions Asked

1. "When exactly did it last work — and did you log off/on, or have you
   been in the same session all along?"
2. "What changed around the time it stopped working?" (probing the
   department transfer and anything done to his account then)

> Note: initial idea of asking the user to "try again tomorrow morning"
> was rejected — it wastes a day and tests nothing we control.

---

## Diagnosis

Checked David's actual group memberships on the DC:

```powershell
Get-ADUser -Identity "dchen" -Properties MemberOf |
    Select-Object -ExpandProperty MemberOf
```

**Result BEFORE fix: EMPTY.** David was in **no groups at all**.

This proved the incident narrative: when David transferred departments,
HR's ticket to add him to SG-Marketing was **never completed**. The
"intermittent" behavior was masking a simple truth — he was never
properly provisioned.

> Diagnostic skill noted: noticing that an output is *missing* something
> expected (empty MemberOf) is evidence, same as a wrong value would be.

---

## Root Cause

Two combined causes:

1. **Missing membership:** David was never added to SG-Marketing during
   his department transfer (incomplete offboarding/onboarding process).
2. **Stale token:** even after fixing membership, group changes do NOT
   apply to an existing Windows session. At logon, Windows builds a
   **Kerberos TGT + access token** containing the group SIDs present *at
   that moment*. File servers check the **token**, not live AD membership.

This is why restarting the *application* didn't help — the app reuses the
same old token. Only a **new logon session** builds a new token.

---

## Fix

```powershell
# 1. Create the group (if absent)
New-ADGroup -Name "SG-Marketing" -GroupScope Global -GroupCategory Security

# 2. Add David (the real fix — what HR's ticket should have done)
Add-ADGroupMember -Identity "SG-Marketing" -Members "dchen"

# 3. Verify membership
Get-ADGroupMember -Identity "SG-Marketing"
Get-ADUser -Identity "dchen" -Properties MemberOf |
    Select-Object -ExpandProperty MemberOf
# Result: CN=SG-Marketing,... present ✅

# 4. Confirm group configuration
Get-ADGroup -Identity "SG-Marketing" |
    Select-Object Name, GroupScope, GroupCategory
# Result: SG-Marketing / Global / Security ✅
```

**Then the user-side action: FULL sign out + sign in** (or reboot).
Not lock screen, not app restart — a new logon session.

---

## Why It "Worked Sometimes" — Explained

| David's story | Explanation |
|---|---|
| Worked yesterday AM | Old session/cached credentials still granted residual access |
| Failed yesterday PM | Old token/cached credentials expired or rejected — with NO groups, he had no granted access |
| Worked today | A new logon built a fresh token that reached resources open to broader groups (Everyone / Authenticated Users) |
| Locked again | A different share/server checked the token specifically against SG-Marketing — which he was not in |

---

## Verification

- ✅ `MemberOf` shows SG-Marketing after the add
- ✅ Group scope/category confirmed (Global / Security)
- ✅ `whoami /groups` demonstrated what a token's group list looks like
- ✅ Instructed user: full logoff/logon, then test the folder

⚠️ Note: `whoami /groups` was run on the DC as Administrator, so it showed
the admin's token (Domain Admins, etc.) — in a real case it must be run
on the *affected user's machine after their new logon*, looking for
SG-Marketing in the list.

---

## Key Concept: M365 / Entra ID Mapping

In Entra ID, group changes don't need a reboot — but they are not always
instant for already signed-in sessions:

- Access tokens are refreshed; SharePoint/OneDrive permission changes can
  take **up to a few hours** to propagate, or apply immediately after the
  user signs out/in of Office apps
- Standard user guidance: *"The change is done on our side — give it
  2–3 hours, or sign out and back into Office apps, then try again."*

On-prem: token = fixed until new logon. Cloud: token = refreshable.
That difference is the takeaway.

---

## Notes / Dead Ends

- First triage idea ("try again tomorrow morning") rejected — poor practice.
- `whoami /groups` executed as Administrator, not the affected user —
  valid as a demo but noted for correctness.
- The intermittent symptom initially suggested a complex problem; the
  actual cause was a simple provisioning gap hidden by session timing.

---

## Lessons Learned

1. **"Sometimes works" = check what changed between attempts** — session
   state, token age, or which server performed the check.
2. **Group membership changes require a new logon session** on-prem —
   the fix isn't done until the session is new.
3. **An empty result is evidence too** — an empty MemberOf list exposed
   the incomplete HR transfer ticket.
4. **Incomplete onboarding/transfer processes are a top source of access
   tickets** — the technical fix is easy; catching the process gap is
   the real value.

---