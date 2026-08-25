# KB‑002 — New‑hire onboarding checklist

**Applies to:** Active Directory account provisioning and dependent onboarding tasks

**Audience:** Tier‑1 / Tier‑2 help desk

**Related ticket:** [HD‑002](../tickets/HD-002-new-hire-onboarding.md)

## Before you start

Confirm the authorized request, full name, department, title, start date, manager, naming
standard, and approvals for nonstandard access. Treat each onboarding component as a separately
verifiable task; AD account creation alone is not complete onboarding.

## Active Directory checklist

- [ ] Find and verify the target OU.

  ```powershell
  Get-ADUser <existing-dept-user> | Select-Object -ExpandProperty DistinguishedName
  ```

- [ ] Create the account disabled and set identity attributes.

  ```powershell
  $ou = 'OU=<Dept>,OU=Departments,OU=Corp,DC=jtwyman,DC=test'
  New-ADUser -Name '<First Last>' -GivenName <First> -Surname <Last> -SamAccountName <user> `
    -UserPrincipalName <user>@jtwyman.test -Path $ou -Enabled $false
  Set-ADUser <user> -Department <Dept> -Title '<Title>' -Company 'JTW Corp'
  ```

- [ ] Add only approved baseline groups; separately verify what their ACLs actually grant.

  ```powershell
  Add-ADGroupMember SG-<Dept> -Members <user>
  Add-ADGroupMember SG-AllStaff -Members <user>
  Get-ADPrincipalGroupMembership <user> | Format-Table Name -Auto
  ```

- [ ] Prompt for a temporary password without putting it in source, shell history, transcripts,
      screenshots, tickets, or email.

  ```powershell
  $temporaryPassword = Read-Host 'Enter temporary password' -AsSecureString
  Set-ADAccountPassword <user> -Reset -NewPassword $temporaryPassword
  Set-ADUser <user> -ChangePasswordAtLogon $true
  ```

- [ ] Enable only when ready, then verify account flags.

  ```powershell
  Enable-ADAccount <user>
  Get-ADUser <user> -Properties Enabled,PasswordExpired |
    Format-Table Name,Enabled,PasswordExpired -Auto
  ```

`PasswordExpired = True` supports the expected first-logon change flag; it does not prove a
successful sign-in or secure credential delivery.

## Credential exposure response

If a temporary password appears in code, documentation, a ticket, chat, email, screenshot, or
other unauthorized location, treat it as compromised even if it was intended to be temporary:

1. Disable the account (or otherwise block authentication per policy).
2. Reset to a new, unique value using `Read-Host -AsSecureString` or the approved identity tool.
3. Preserve the required first-logon change setting.
4. Re-enable only when authorized and ready for a new approved handoff.
5. Remove the exposed value from current documentation and follow repository/history and incident
   procedures as required. Removal alone does not invalidate the exposed credential.

Never claim remediation or secure handoff without evidence from the relevant system/process.

## Complete onboarding workstreams

Track and verify independently, as applicable:

- [ ] mailbox and licensing;
- [ ] distribution lists and application roles;
- [ ] effective share/application permissions (not group membership alone);
- [ ] standard software and device preparation;
- [ ] approved credential handoff with no secret recorded in the ticket; and
- [ ] first sign-in/readiness confirmation.

## Rules of thumb

- Least privilege: baseline roles only; nonstandard access requires resource-owner approval.
- Keep the account disabled until the authorized onboarding state is ready.
- Distinguish "provisioning executed" from "onboarding complete."

## Offboarding

Follow retention and legal policy: disable access promptly, reset/revoke credentials and sessions,
remove entitlements, and reassign/retain data. Do not delete an account until policy permits it.
