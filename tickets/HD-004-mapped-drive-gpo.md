# HD‑004 — Mapped drive (S:) missing after login

| | |
|---|---|
| **Ticket #** | 496388 |
| **Reported by** | Marcus Lee (mlee), IT |
| **Help topic** | Report a Problem |
| **Priority** | Normal |
| **Classification** | Group Policy configuration diagnostic |
| **Exercise outcome** | Diagnostic only; root cause and endpoint remediation unverified |
| **Captured osTicket state** | Open / Unassigned |

## Summary

A user reports that an `S:` drive normally mapped at login by Group Policy is missing while
nearby colleagues still have theirs.

> *"My S: drive that group policy usually maps at login stopped appearing. Coworkers nearby
> still have theirs. I need it for shared tools." — Marcus Lee, IT*

## Impact & priority

One user reports loss of a shared resource but can otherwise work → **Normal**. Other users being
unaffected makes a user-, device-, or policy-scope issue plausible, but does not by itself rule
out all share, DFS, network, or drive-letter conditions.

## Configuration evidence captured

### 1. GPO inventory

```powershell
Get-GPO -All | Format-Table DisplayName,GpoStatus -Auto
```

```text
Workstation Security Baseline     AllSettingsEnabled
Default Domain Policy             AllSettingsEnabled
IT Department Marker Policy       AllSettingsEnabled
Default Domain Controllers Policy AllSettingsEnabled
Domain Logon Banner               AllSettingsEnabled
```

*(Evidence: `evidence/screenshots/12-gpo-inventory.png`)*

### 2. Permissions on a candidate policy

```powershell
Get-GPPermission -Name 'IT Department Marker Policy' -All |
  Format-Table @{n='Trustee';e={$_.Trustee.Name}},TrusteeType,Permission -Auto
```

```text
Trustee                       Permission
-------                       ----------
SG-IT                         GpoApply
Domain Admins                 GpoEditDeleteModifySecurity
Enterprise Admins             GpoEditDeleteModifySecurity
ENTERPRISE DOMAIN CONTROLLERS GpoRead
SYSTEM                        GpoEditDeleteModifySecurity
```

*(Evidence: `evidence/screenshots/13-gpo-security-filtering.png`)*

This proves that `SG-IT` has `GpoApply` on **IT Department Marker Policy**. It does not prove that
the policy contains an `S:` drive-map preference or that it is linked to an OU in `mlee`'s user
or computer scope.

## Diagnostic assessment

A security-filtering or scope problem is a reasonable hypothesis, but no root cause is proven.
The available evidence does not include:

* the GPO report showing drive-map content and item-level targeting;
* the relevant OU link, inheritance, enforcement, or block-inheritance state;
* `mlee`'s current/before group membership or evidence of removal/re-addition;
* client-side `gpresult`/RSoP showing applied or denied GPOs and denial reasons; or
* verification that `S:` was remapped and reachable.

## Next diagnostic steps

```powershell
# Server-side: inspect GPO content and links
Get-GPOReport -Name 'IT Department Marker Policy' -ReportType Html -Path .\IT-Policy.html
Get-ADUser mlee -Properties MemberOf | Select-Object -ExpandProperty MemberOf
```

On the affected client, collect `gpresult /h` (or Group Policy Results), inspect the Group Policy
operational log, confirm the expected drive-map item and target path, check for an existing `S:`
assignment, and test the UNC path separately. Do not change group membership until the intended
entitlement and approval are verified.

## Communication to the user

> Hi Marcus, we confirmed that `SG-IT` can apply a candidate IT policy, but we have not yet shown
> that this policy maps `S:` or why it did not apply to your session. We need client policy results
> and the GPO's drive-map/link configuration before making a change. — IT Help Desk

## Escalation decision

Escalate to the Group Policy/file-services owner if the help desk cannot inspect the GPO links,
drive-map preference, effective client policy, or share path. This case remains a configuration
diagnostic and must not be marked resolved without client-side and resource-access verification.
