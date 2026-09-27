TKT-005-printer-deescalation.md

```markdown
# TKT-005 — Recurring Printer Failure + Angry User De-escalation (Maria Gonzalez)

**Date:** 26/09/2026
**Priority:** High (per user) — triaged to Medium after analysis
**SLA Target:** 5-minute response (angry repeat caller)
**Status:** ✅ RESOLVED
**Technician:** yousef
**Environment:** TAILWIND-DC1 (print spooler lab) — printer: TAILWIND-PRINT01, shared by 8 Sales users

---

## Symptom

Maria Gonzalez (Sales) called, extremely angry — **third ticket in one week**
about the same printer:

- "Third time I'm calling about this printer!"
- Previous fix lasted about an hour before it broke again
- Has a sales report to print before a 3 PM meeting
- Threatened to escalate to the manager
- Frustrated at repeating her story to a different person each call

**Ticket history on TAILWIND-PRINT01:**
1. Earlier this week — "print queue stuck" → resolved by clearing queue
2. Yesterday — "printer offline" → resolved by restarting Print Spooler
3. Today — same problem again (this ticket)

---

## 🧠 De-escalation: Opening Script (First 30 Seconds)

> *"Maria, I'm sorry — third time in one week is absolutely frustrating,
> and I can see your previous tickets right here, so you don't need to
> explain it all again. I have 25 minutes before your meeting and I'm
> going to use every one of them to get this fixed properly."*

**Techniques used:**
- Acknowledged frustration WITHOUT "calm down" (never works)
- No defensiveness about the previous technician
- Proved I read her history — she doesn't repeat herself
- Anchored on her real pressure: the 3 PM meeting

---

## Triage Analysis

**Key evidence: 8 people share this printer — only Maria reported it today.**

Deduction: the printer is (mostly) fine — the problem is **client-side**,
on Maria's computer. Her machine is most likely sending a **corrupt or
problematic print job** that jams the shared queue. This also explains the
pattern: "fixed, then broke again an hour later" = the moment Maria printed
again, her bad job re-jammed the queue.

This re-frames priority too: user says "High," but with one affected user
and a workaround available, the business impact is Medium — while the
*response* urgency stays High because she's an angry repeat caller with a
deadline.

---

## Diagnosis

The Print Spooler queues jobs in `C:\Windows\System32\spool\PRINTERS`.
A corrupt job file in that folder causes repeated queue jams:

- Restarting the spooler **re-loads the corrupt file** → problem returns
- Clearing the queue *without removing the file* → problem returns
- Only a full clean (stop → delete files → start) removes the cause

---

## Fix — Full Spooler Clean (Correct Order)

```powershell
# 1. Check spooler status
Get-Service -Name Spooler | Select-Object Name, Status, StartType

# 2. Inspect the job folder (where stuck files pile up)
Test-Path "C:\Windows\System32\spool\PRINTERS"
Get-ChildItem "C:\Windows\System32\spool\PRINTERS" -ErrorAction SilentlyContinue

# 3. THE REAL FIX — stop, clear, start (ORDER MATTERS:
#    files can only be deleted while the service is stopped)
Stop-Service -Name Spooler -Force
Remove-Item -Path "C:\Windows\System32\spool\PRINTERS\*" -Force -ErrorAction SilentlyContinue
Start-Service -Name Spooler

# 4. Verify
Get-Service -Name Spooler | Select-Object Name, Status
# Result: Spooler — Running ✅
```

---

## Root Fix vs Symptom Fix (Key Concept)

- **Restarting the Spooler alone = SYMPTOM fix:** on restart, the service
  re-loads whatever is in the queue folder — including the corrupt job —
  so the failure comes back as soon as printing resumes.
- **Clearing `spool\PRINTERS\*` while stopped = ROOT fix:** the corrupt
  file (the cause) is physically removed, so the service comes up clean.

This is why the two previous tickets "fixed" the problem without solving it.

---

## Recurrence Prevention (What the Previous Techs Skipped)

After 3 identical tickets, the question changes from "how do I fix it?"
to "why does it KEEP happening?"

1. **Investigate the jobs:** check which documents jam the queue
   (huge PDFs / photo-heavy files are classic spooler killers) and verify
   Maria's **printer driver** on her PC matches the print server's version —
   a mismatched/outdated driver is the most common source of corrupt jobs.
2. **Prevent:** update the driver on her machine; recommend the team use
   the shared print-server driver rather than direct-IP printing.
3. **Escalation rule:** if this queue jams a **4th time**, escalate for a
   driver/firmware change — do not just clean it a 4th time.

---

## De-escalation: Callback Script (After the Fix)

> *"Hi Maria, it's IT again — good news. The problem wasn't the printer
> itself: a corrupted print file on your computer was jamming the print
> queue. The previous fixes cleared the jam but left the bad file behind,
> which is why it kept coming back. This time I removed the file itself,
> so it's fixed at the source. If anything happens again before your
> 3 PM — send the report to the printer near reception as a backup, and
> call me straight away."*

**Required elements included:** plain-English explanation (no jargon),
why THIS fix differs from the previous two, and a one-click fallback plan
for the pre-meeting window.

---

## Verification

- ✅ Spooler service: **Running**, StartType Automatic
- ✅ Spool folder cleared while service stopped
- ✅ User informed of fix + fallback plan before her deadline

---

## M365 Mapping

In a Microsoft 365 environment, the equivalent pattern is **Universal
Print** — print queues are cloud-managed, no on-prem print server. The
same triage logic applies: if only ONE user can't print to a shared
cloud printer, check that user's client/driver first; if everyone fails,
check the printer/connector. Fix-the-cause discipline is identical:
clearing a stuck cloud job without fixing the source client = the
jam returns.

---

## Notes / Dead Ends

- First draft of opening line ("OK, let me check your history...") lacked
  empathy and deadline acknowledgment — revised into the full script above.
- Initial triage idea included asking the user to test cables/WiFi;
  refined after the "only 1 of 8 users affected" deduction pointed
  client-side first.
- Communication is a deliverable in this ticket, equal in weight to the
  PowerShell fix — a technically perfect fix delivered badly still loses
  the customer.

---

## Lessons Learned

1. **Repeat tickets = process failure, not bad luck.** Three identical
   tickets means the root cause was never addressed.
2. **Scope first:** one affected user out of eight → suspect the client,
   not the shared resource.
3. **Stop → clear → start, in that order.** The spooler reloads corrupt
   jobs on restart; deletion only works while stopped.
4. **De-escalation formula:** acknowledge frustration + show you read the
   history + anchor on their deadline + give a fallback plan.
5. **Every fix ends with a prevention step** — that's the difference
   between a tech who closes tickets and one who stops them.

---