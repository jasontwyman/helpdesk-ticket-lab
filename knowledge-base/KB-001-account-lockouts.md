# KB‑001 — Account lockouts: find the source and unlock

**Applies to:** Active Directory domain accounts

**Audience:** Tier‑1 / Tier‑2 help desk

**Related ticket:** [HD‑001](../tickets/HD-001-account-lockout.md)

## When to use this

A user reports that a domain account is locked or repeatedly becomes locked.

## 1. Confirm account state

```powershell
Search-ADAccount -LockedOut | Format-Table Name,SamAccountName,LockedOut,LastLogonDate -Auto
Get-ADUser <user> -Properties LockedOut,BadLogonCount |
  Format-Table Name,Enabled,LockedOut,BadLogonCount -Auto
```

`BadLogonCount` and the lockout state confirm the symptom; they do not identify the submitting
device or prove whether the user's current password is correct.

## 2. Read Event 4740 by named XML fields

Run on the domain controller that logged the lockout (or query the relevant DCs). Do **not** rely
on positional property indexes. Parse `CallerComputerName` by name:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4740} -MaxEvents 20 |
  ForEach-Object {
    $event = $_
    $data = @{}
    ([xml]$event.ToXml()).Event.EventData.Data | ForEach-Object {
      $data[$_.Name] = $_.'#text'
    }
    [pscustomobject]@{
      TimeCreated        = $event.TimeCreated
      TargetUserName     = $data['TargetUserName']
      CallerComputerName = $data['CallerComputerName']
    }
  } | Where-Object TargetUserName -eq '<user>' | Format-Table -Auto
```

Correlate timestamps and query other DCs if necessary. `CallerComputerName` is a lead, not a
complete root-cause determination; it may be blank, unexpected, proxied, or identify a system
where controlled tests were generated.

## 3. Form and test hypotheses

Common production hypotheses include saved credentials in a phone/mail client, Wi‑Fi profile,
Credential Manager, mapped drive, task/service, or disconnected session. Automated guessing or
malicious activity is another possibility. Investigate the named caller and authentication logs;
do not declare stale credentials without evidence.

## 4. Unlock and verify directory state

```powershell
Unlock-ADAccount -Identity <user>
Get-ADUser <user> -Properties LockedOut | Format-Table Name,LockedOut -Auto
```

This verifies that AD reports the account unlocked. Ask the user to test sign-in separately and
record the result; an unlocked flag does not prove a successful sign-in or validate a password.

## When to escalate

Escalate repeated lockouts, unknown or sensitive caller computers, anomalous times/source IPs,
or activity inconsistent with the user to Security. Escalate DC/event-log gaps or ambiguous
caller attribution to the identity team.

## Prevention

Use a risk-appropriate lockout policy, monitoring, and alerting. Document the actual cause before
changing saved credentials or services, and never weaken lockout controls simply to hide a
repeating authentication problem.
