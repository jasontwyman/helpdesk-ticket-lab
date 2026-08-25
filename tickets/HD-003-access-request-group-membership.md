# HD‑003 — Access request: Finance shared folder

| | |
|---|---|
| **Ticket #** | 615099 |
| **Reported by** | Nina Alvarez (nalvarez), Sales |
| **Help topic** | Account & Access |
| **Priority** | Normal |
| **Classification** | Executed Active Directory exercise |
| **Exercise outcome** | Membership exercise completed and reverted; effective access unverified |
| **Captured osTicket state** | Open / Unassigned |

## Summary

A Sales employee requests read access to the Finance shared folder for a cross‑department Q3
budget project. The request states that Grace Chen in Finance approved it.

> *"I'm on the Q3 cross‑department budget project and need access to the Finance shared folder.
> Grace Chen in Finance has approved." — Nina Alvarez, Sales*

## Impact & priority

Business‑enabling, not an outage → **Normal**. Cross‑department access requires verified
resource-owner approval and the narrowest available entitlement.

## Approval and evidence boundary

Grace Chen's approval is **scenario context supplied in the request**; the repository does not
contain an independent approval record. In production, pause until approval is verified through
the approved workflow and linked to the ticket.

The captured before state lists three members of `SG-Finance`:

```powershell
'--- BEFORE ---'; Get-ADGroupMember SG-Finance | Format-Table Name,SamAccountName -Auto
```

```text
Grace Chen   gchen
Omar Hassan  ohassan
Laura Wells  lwells
```

## Change executed

```powershell
Add-ADGroupMember -Identity SG-Finance -Members nalvarez
'--- AFTER ---'; Get-ADGroupMember SG-Finance | Format-Table Name,SamAccountName -Auto
```

```text
Nina Alvarez nalvarez
Grace Chen   gchen
Omar Hassan  ohassan
Laura Wells  lwells
```

*(Evidence: `evidence/screenshots/08-access-request-group-before-after.png`)*

The evidence demonstrates the AD membership change only. It does **not** show the share or NTFS
ACLs, prove that `SG-Finance` grants read-only access, or demonstrate a refreshed user token and
successful access.

## Least‑privilege caveat

`SG-Finance` appears to be a broad departmental group. Adding a Sales user may grant more than
the requested project-specific read access. Before using this method in production, verify the
share and NTFS ACLs and prefer a dedicated, read-only, project-scoped group when available. Do
not describe this membership as least privilege until effective permissions are confirmed.

## Communication to the user

> Hi Nina, your account was added to `SG-Finance` in this scenario. Effective access and its
> permission level have not yet been verified. We also need the resource-owner approval retained
> in the ticket and a defined removal date before treating the request as complete. — IT Help Desk

## Escalation decision

Route the request to the Finance data owner or access-governance team if approval cannot be
independently verified, if `SG-Finance` is broader than required, or if effective ACLs are
unclear.

## Review / removal

The broad `SG-Finance` membership was removed during post-review lab cleanup rather than left as
standing cross-department access. A verification query shows `nalvarez` only in `Domain Users`,
`SG-Sales`, and `SG-AllStaff`; `SG-Finance` is absent.

*(Evidence: `evidence/screenshots/18-ad-remediation-cleanup.png`)*

This cleanup does not retroactively prove approval, read-only access, or effective permissions.
For a real request, record a specific owner and review/removal date before granting access.
