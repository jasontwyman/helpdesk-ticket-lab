# Evidence Manifest

This index covers all **18 tracked screenshots**. SHA-256 values identify the exact files in this
working tree. Classification means:

- **Executed** — supports a narrow action or queried state from one of the three AD exercises.
- **Config** — supports environment, platform, policy, or ticketing configuration/state only.
- **Illustrative** — shows synthetic case-study content, not execution of the narrated workflow.

A screenshot is a point-in-time visual artifact, not an independent audit log. “No visible
redaction” does not establish an untouched original; where provenance is unavailable, edit history
is explicitly recorded as unverified. The manifest intentionally contains no credential value.

| # | Screenshot | SHA-256 | Dimensions | Class | Supported claim | Redaction / edit status | Limitations |
|---|---|---|---|---|---|---|---|
| 01 | [AD environment users](screenshots/01-ad-environment-users.png) | `e0f5f647e721ab9d2300a962c1e4ab3fd3ed933085918b37e5027aa9b9691159` | 1024×768 | Config | A PowerShell query visibly lists synthetic AD users with selected account fields. | No visible redaction; provenance/edit history unverified. | Does not prove creation steps, credential state, sign-in, or effective access. |
| 02 | [AD OUs and security groups](screenshots/02-ad-ou-and-security-groups.png) | `f8ae5785f2c7a93f51be0c79def7c9d9aa8d1b99f09f2fc184c0dc2441468f1b` | 1024×768 | Config | A PowerShell query visibly lists security groups in the lab directory. | No visible redaction; provenance/edit history unverified. | Does not prove group membership, ACLs, purpose, or effective permissions. |
| 03 | [Lockout policy enabled](screenshots/03-lockout-policy-enabled.png) | `50f1547a45bd3e20cfd33e9591930276be67f1e0b041852eda3b9bf109eb6aa2` | 1024×768 | Config | Visible commands set the domain lockout threshold to 5 and query that threshold. | No visible redaction; provenance/edit history unverified. | The capture shows console input/state only; it does not show prior value, full policy, client application, or restoration. |
| 04 | [Controlled failed authentication attempts](screenshots/04-lockout-failed-auth-attempts.png) | `781bbe65efdc44849c5a1dd47827ca0e3d5f0dbd0aab94cb6c56dc3c9360380e` | 1024×768 | Executed | Repeated `Get-ADUser` attempts visibly return rejected-credential errors in the controlled lab. | No visible redaction; provenance/edit history unverified. No credential value is visible. | Errors alone do not prove final lockout state, event generation, or a production-source cause. |
| 05 | [Account locked out](screenshots/05-account-locked-out.png) | `73857afac5606ca535bde8a5eb22226343be9c46613833e5bf4cef5cd0cdf7e0` | 1024×768 | Executed | AD query output visibly shows synthetic `sconnor` with `LockedOut = True` and a bad-logon count. | No visible redaction; provenance/edit history unverified. | Does not prove caller, root cause, timing chain, or user impact. |
| 06 | [Lockout source Event 4740](screenshots/06-lockout-source-event-4740.png) | `d9c5ae6dc205f3a24776a39fc22bf895419d44310ae4563b74a06a9e184ca2b2` | 1024×768 | Executed | Parsed Security Event 4740 fields visibly identify the locked account and `DC01` as caller. | No visible redaction; provenance/edit history unverified. | A single parsed event screenshot is not raw EVTX and does not establish why the attempts occurred. |
| 07 | [Account unlocked](screenshots/07-account-unlocked.png) | `7b3d89f44623940930efe66f084caface6d5312e91e499a7a2a32c649a279494` | 1024×768 | Executed | Post-action AD output visibly shows `LockedOut = False` and zero locked accounts remaining. | No visible redaction; provenance/edit history unverified. | Does not prove successful sign-in, credential validity, or removal of a recurring source. |
| 08 | [Access group before and after](screenshots/08-access-request-group-before-after.png) | `836732c8801f3ee6f9e4af1a3e3c64c84249ca3a505c7851c51eee4076c5abc3` | 1024×768 | Executed | Console output visibly shows `SG-Finance` membership before and after adding synthetic `nalvarez`. | No visible redaction; provenance/edit history unverified. | Does not prove approval, ACL scope, least privilege, token refresh, or share access. |
| 09 | [Onboarding user created](screenshots/09-onboarding-user-created.png) | `932ac0c83c0c155de47a93d8e67cc1ee6267da5a0d9b4bf11436b4563c4a40e8` | 1024×768 | Executed | Visible commands/query show creation of synthetic `kross` and selected identity attributes while disabled. | No visible redaction; provenance/edit history unverified. | Does not prove all onboarding workstreams, credential setup, or successful use. |
| 10 | [Onboarding group membership](screenshots/10-onboarding-group-membership.png) | `0e83e199efeff56d4d7f772a08f550e4252202f48776d1febcf0e8bbdd0e6e71` | 1024×768 | Executed | Query output visibly lists `kross` in Domain Users, `SG-Finance`, and `SG-AllStaff`. | No visible redaction; provenance/edit history unverified. | Group names do not prove resource ACLs or effective access. |
| 11 | [Onboarding enabled / must change](screenshots/11-onboarding-enabled-mustchange.png) | `171787a3cd96920b6a66000ed85cb2aca9720f0292e0f432f37b960d65df8435` | 1024×768 | Executed | Post-action output visibly shows synthetic `kross` enabled with `PasswordExpired = True` at capture time. | **Edited:** an opaque banner replaces the prior temporary-password command area; the credential is not present in this file. | Redaction removes command-level provenance; does not prove secure generation/handoff, first sign-in, or later state. Screenshot 18 records later disable/reset cleanup. |
| 12 | [GPO inventory](screenshots/12-gpo-inventory.png) | `a971cb353d22731f910da2dd3cd1c853bf535aee1a871f3dbe6110ba5377a314` | 1024×768 | Config | `Get-GPO -All` output visibly lists five GPO names and enabled status. | No visible redaction; provenance/edit history unverified. | Inventory does not prove links, settings, security scope, endpoint application, or health. |
| 13 | [GPO security filtering](screenshots/13-gpo-security-filtering.png) | `340ddbc03bc31c188f6be3be89c0af7adf78580ded5eb0fc00f707f3b5251724` | 1024×768 | Config | Visible permission output shows `SG-IT` with `GpoApply` on `IT Department Marker Policy`. | No visible redaction; provenance/edit history unverified. | Does not prove the policy maps `S:`, is linked/in scope, applied on a client, or caused the report. |
| 14 | [osTicket ticket queue](screenshots/14-osticket-ticket-queue.jpg) | `fe24644e74635cd83bf5c2a680914b0c889df56cac73d43e4599e0c0d7bb4baf` | 1568×745 | Illustrative | The synthetic queue visibly contains seven records in the **Open** view with blank assignees. | No visible redaction; provenance/edit history unverified. | Seeded records do not prove investigation, replies, assignment, escalation, resolution, closure, or SLA performance. |
| 15 | [osTicket ticket detail](screenshots/15-osticket-ticket-detail.jpg) | `65274f20fce31ac847a351a944fc496496c96be874447a8b2865a38f6870c34e` | 1568×745 | Illustrative | A synthetic account-lockout ticket is visibly **Open** and **Unassigned** with seeded user text. | No visible redaction; provenance/edit history unverified. | Does not show an agent response, AD action, assignment, status transition, or resolution. |
| 16 | [osTicket help topics](screenshots/16-osticket-help-topics-config.jpg) | `7c752e27bec5d916b028d8809505ce4e4cae76ba61b99414e3e6586d4c39abdb` | 1568×745 | Config | Admin view visibly lists configured help topics and selected routing/default fields. | No visible redaction; provenance/edit history unverified. | Does not prove ticket handling, permissions, notification delivery, routing behavior, or SLA execution. |
| 17 | [osTicket client portal](screenshots/17-osticket-client-portal.jpg) | `c168c4e9663869d82d0adc21524ac7d3d72162a19d0d17c129f54bf1d05449ae` | 1568×745 | Config | The client portal visibly renders an open-new-ticket form and help-topic selector. | No visible redaction; provenance/edit history unverified. | Rendering does not prove successful submission, mail flow, authentication, HTTPS, or external availability. |
| 18 | [AD remediation cleanup](screenshots/18-ad-remediation-cleanup.png) | `b4715d58752545331a230506d8344761dea5e2364116f818d9ad9961e788e90f` | 970×440 | Executed | Visible commands and queries show a reset using an undisclosed variable, `kross` disabled, and `nalvarez` absent from the listed post-cleanup groups. | No visible redaction; provenance/edit history unverified. The replacement value is not visible. | Does not prove replacement-value generation, secure handoff, complete historical state, approval, or effective resource access. |

## Non-screenshot configuration evidence

[`config/helpdesk-vm-network.txt`](config/helpdesk-vm-network.txt) preserves sanitized host-side
VirtualBox output showing NIC 1 disabled and NIC 2 attached to `intnet`. It does not prove the
guest route table, service reachability, firewall state, or production-grade isolation.

## Integrity check

From the repository root, verify the indexed screenshot hashes without changing the artifacts:

```bash
sha256sum evidence/screenshots/*
```

Compare all 18 values to this manifest. Hash equality proves byte identity with this index only; it
does not establish capture provenance or the truth of activity outside the visible frame.
