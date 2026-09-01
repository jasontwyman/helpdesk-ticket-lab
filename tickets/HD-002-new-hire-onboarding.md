# HD‑002 — New‑hire setup (Finance)

| | |
|---|---|
| **Ticket #** | 177196 |
| **Reported by** | Elena Garcia (egarcia), HR |
| **Help topic** | New Hire Onboarding |
| **Priority** | Normal |
| **Classification** | Executed Active Directory exercise |
| **Exercise outcome** | Provisioning executed; exposed credential remediated and account disabled |
| **Captured osTicket state** | Open / Unassigned |

## Summary

HR requests a domain account for a new Financial Analyst, **Kevin Ross**, starting Monday. He
needs access to Finance and standard staff resources. Hiring manager: Grace Chen (Finance).

> *"Please set up a domain account for our new Financial Analyst, Kevin Ross, starting Monday.
> He needs the Finance shared drive plus standard staff access. Hiring manager: Grace Chen." —
> Elena Garcia, HR*

## Impact & priority

Planned work with a Monday deadline, not an outage → **Normal**. This case documents the Active
Directory provisioning that was executed; it does not evidence the entire onboarding workflow.

## Investigation / preparation

```powershell
Get-ADUser gchen | Select-Object -ExpandProperty DistinguishedName
# CN=Grace Chen,OU=Finance,OU=Departments,OU=Corp,DC=jtwyman,DC=test
```

Target OU: `OU=Finance,OU=Departments,OU=Corp,DC=jtwyman,DC=test`.
The scenario uses `SG-Finance` and `SG-AllStaff`; group names alone do not prove the effective
share permissions they provide.

## Provisioning executed

**1. Create the account disabled and set identity attributes.**

```powershell
$ou = 'OU=Finance,OU=Departments,OU=Corp,DC=jtwyman,DC=test'
New-ADUser -Name 'Kevin Ross' -GivenName Kevin -Surname Ross -SamAccountName kross `
  -UserPrincipalName kross@jtwyman.test -Path $ou -Enabled $false
Set-ADUser kross -Department Finance -Title 'Financial Analyst' -Company 'JTW Corp'
```

*(Evidence: `evidence/screenshots/09-onboarding-user-created.png`)*

**2. Add the requested baseline groups and capture membership.**

```powershell
Add-ADGroupMember SG-Finance  -Members kross
Add-ADGroupMember SG-AllStaff -Members kross
Get-ADPrincipalGroupMembership kross | Format-Table Name -Auto
# Domain Users / SG-Finance / SG-AllStaff
```

*(Evidence: `evidence/screenshots/10-onboarding-group-membership.png`)*

**3. Prompt securely for a temporary password, require change at first logon, and enable.**

```powershell
$temporaryPassword = Read-Host 'Enter temporary password' -AsSecureString
Set-ADAccountPassword kross -Reset -NewPassword $temporaryPassword
Set-ADUser kross -ChangePasswordAtLogon $true
Enable-ADAccount kross
Get-ADUser kross -Properties Enabled,PasswordExpired |
  Format-Table Name,SamAccountName,Enabled,PasswordExpired -Auto
```

```text
Name       SamAccountName Enabled PasswordExpired
----       -------------- ------- ---------------
Kevin Ross kross             True            True
```

The capture shows the account enabled and flagged for a password change at next logon.
*(Evidence: `evidence/screenshots/11-onboarding-enabled-mustchange.png`)*

## Credential exposure remediation

A temporary password had previously been embedded in this case study and public screenshot. It
was treated as exposed. Removing it from documentation was necessary but not sufficient.

Post-review remediation was executed on DC01:

1. The exposed value was replaced through `Set-ADAccountPassword` using an undisclosed
   `SecureString` variable; the replacement value is not present in the repository or evidence.
2. `kross` was disabled.
3. The temporary `nalvarez` Finance membership from HD-003 was also removed during cleanup.

The preserved screenshot shows the reset invocation without a visible error and shows `kross`
with `Enabled = False`; it does not independently verify the replacement value or how it was
generated. Before any future enablement, IT must set a new approved temporary credential, restore
`ChangePasswordAtLogon = True`, and complete a secure handoff without recording the value.

*(Evidence: `evidence/screenshots/18-ad-remediation-cleanup.png`)*

No secure credential handoff or successful first sign-in is claimed.

## Evidence boundaries

The screenshots support AD object creation, listed group membership, and the enabled/
password-change flags. They do **not** independently prove:

* effective read/write access to the Finance share;
* mailbox creation, licensing, distribution lists, software, or device setup;
* delivery of a temporary credential through a secure channel; or
* a successful first sign‑in.

## Communication to the requester

> Hi Elena, the documented Active Directory provisioning for Kevin (`kross`) was executed.
> Because a temporary credential was exposed in the case documentation, IT replaced it and
> disabled the account. The account must remain disabled until a new approved temporary
> credential and secure handoff are prepared.
> Mailbox, device, licensing, and effective resource access require separate verification. — IT
> Help Desk

## Escalation decision

Standard AD provisioning does not require escalation, but the exposed temporary credential
requires immediate account disable/reset under the credential‑exposure process. Nonstandard
access still requires resource-owner approval.

## Follow‑up

Use **[KB‑002 — New‑hire onboarding checklist](../knowledge-base/KB-002-new-hire-onboarding-checklist.md)**
to complete and evidence each remaining onboarding component.
