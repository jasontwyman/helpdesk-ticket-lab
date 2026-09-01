# HD‑001 — Locked out of my account

| | |
|---|---|
| **Ticket #** | 723333 |
| **Reported by** | Sarah Connor (sconnor), HR |
| **Help topic** | Account & Access |
| **Priority** | High |
| **Classification** | Executed Active Directory exercise |
| **Exercise outcome** | Account unlocked; successful user sign-in not verified |
| **Captured osTicket state** | Open / Unassigned |

## Summary

User cannot sign in to her workstation; Windows reports the account is locked. She has a
payroll deadline the same day, so business impact is immediate.

> *"Hi, I can't log into my computer this morning – it says my account is locked. I tried my
> password a few times. I have a payroll deadline today and need access ASAP." — Sarah Connor, HR*

## Impact & priority

A single locked user, but time‑sensitive (payroll) and blocking the user completely →
**High**. No wider outage was reported; only `sconnor` was identified as affected.

## Investigation

**1. Confirm the lockout and gather account facts.**

```powershell
Search-ADAccount -LockedOut | Format-Table Name,SamAccountName,LockedOut,LastLogonDate -Auto
Get-ADUser sconnor -Properties LockedOut,BadLogonCount |
    Format-Table Name,SamAccountName,Enabled,LockedOut,BadLogonCount -Auto
```

The captured result shows the account locked with a bad‑logon count of 5:

```text
Name          SamAccountName Enabled LockedOut BadLogonCount
----          -------------- ------- --------- -------------
Sarah Connor  sconnor           True      True             5
```

*(Evidence: `evidence/screenshots/05-account-locked-out.png`)*

**2. Find where the lockout came from.** The domain controller records Security Event ID 4740
when an account is locked. The retained screenshot used positional `Properties[0]` and
`Properties[1]` projections to label the target and caller. Because positional ordering is easy
to misinterpret, treat that capture as a display of the selected event rather than robust parser
provenance. For repeatable analysis, parse the event XML by field name:

```powershell
$event = Get-WinEvent -FilterHashtable @{LogName='Security';Id=4740} -MaxEvents 1
$eventData = @{}
([xml]$event.ToXml()).Event.EventData.Data | ForEach-Object {
    $eventData[$_.Name] = $_.'#text'
}
[pscustomobject]@{
    TimeCreated    = $event.TimeCreated
    LockedAccount  = $eventData['TargetUserName']
    SourceComputer = $eventData['CallerComputerName']
} | Format-List
```

```text
TimeCreated    : 8/22/2026 6:16:54 PM
LockedAccount  : sconnor
SourceComputer : DC01
```

*(Evidence: `evidence/screenshots/06-lockout-source-event-4740.png`; the screenshot preserves the
older positional projection, while the safer named-field parser is documented above.)*

## Root cause

This lab lockout was caused by **controlled bad‑password attempts generated on `DC01`**, which
drove the account to the configured lockout threshold. The evidence does not establish that a
phone, mapped drive, saved credential, or other production endpoint caused this event.

In production, stale credentials are a common **hypothesis** to investigate after reading
`CallerComputerName`; they are not the demonstrated cause of this lab case. An unexpected or
unknown source can also indicate automated guessing or malicious activity.

## Resolution

```powershell
Unlock-ADAccount -Identity sconnor
Get-ADUser sconnor -Properties LockedOut | Format-Table Name,Enabled,LockedOut -Auto
'Locked accounts remaining: ' + (Search-ADAccount -LockedOut).Count
```

```text
Name          Enabled LockedOut
----          ------- ---------
Sarah Connor     True     False

Locked accounts remaining: 0
```

*(Evidence: `evidence/screenshots/07-account-unlocked.png`)*

The evidence confirms that the account was unlocked. It does **not** independently demonstrate
a successful user sign‑in, validate the user's password, or establish that saved credentials
were corrected.

## Communication to the user

> Hi Sarah, the directory now shows your account as unlocked. Please try signing in again. If it
> locks again, note the time and contact the Help Desk so we can correlate the next lockout event
> with its caller computer. — IT Help Desk

## Escalation decision

No escalation was required for the controlled lab lockout and unlock. In production, escalate
repeated lockouts, events from an unknown or sensitive host, or activity inconsistent with the
user to Security for investigation.

## Prevention / follow‑up

* Confirm and document the account‑lockout policy; the lab evidence shows a threshold of 5 in
  `evidence/screenshots/03-lockout-policy-enabled.png`.
* If a production lockout repeats, use the named `CallerComputerName` value to investigate saved
  credentials, services, tasks, sessions, or malicious attempts without assuming a cause.
* Reusable procedure: **[KB‑001](../knowledge-base/KB-001-account-lockouts.md)**.
