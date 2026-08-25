# Verification Matrix

This one-page matrix separates what was performed from what is only described. “Verified” means
the cited repository artifact supports the narrow claim shown; it is not a production-readiness
claim. All preserved osTicket case records are **Open / Unassigned**.

| Case | Classification | Performed or evaluated | Verification evidence | Verified outcome | Explicit gap / next proof |
|---|---|---|---|---|---|
| [HD-001](../tickets/HD-001-account-lockout.md) | **Executed AD exercise** | Generated controlled failed authentication attempts, reviewed lockout state and Event 4740, then unlocked the account. | [03](../evidence/screenshots/03-lockout-policy-enabled.png)–[07](../evidence/screenshots/07-account-unlocked.png) | Directory query shows `sconnor` unlocked; Event 4740 parsing shows `DC01` as caller. | No successful endpoint sign-in or corrected saved-credential test. |
| [HD-002](../tickets/HD-002-new-hire-onboarding.md) | **Executed AD exercise** | Created synthetic `kross`, assigned listed groups, set first-logon password-change state, then remediated the exposed credential and disabled the account. | [09](../evidence/screenshots/09-onboarding-user-created.png)–[11](../evidence/screenshots/11-onboarding-enabled-mustchange.png), [18](../evidence/screenshots/18-ad-remediation-cleanup.png) | AD queries show object attributes/membership at exercise time and `Enabled = False` after cleanup. | No credential handoff, first sign-in, mailbox/device setup, or effective share-access test. |
| [HD-003](../tickets/HD-003-access-request-group-membership.md) | **Executed AD exercise** | Added synthetic `nalvarez` to `SG-Finance`, captured before/after membership, then removed it during cleanup. | [08](../evidence/screenshots/08-access-request-group-before-after.png), [18](../evidence/screenshots/18-ad-remediation-cleanup.png) | AD output shows the membership change and later absence from the listed memberships. | Approval, ACL scope, read-only effect, refreshed token, and share access are unverified. |
| [HD-004](../tickets/HD-004-mapped-drive-gpo.md) | **GPO configuration diagnostic** | Inspected GPO inventory and permissions on a candidate policy. | [12](../evidence/screenshots/12-gpo-inventory.png), [13](../evidence/screenshots/13-gpo-security-filtering.png) | Captures show listed GPOs and `SG-IT` with `GpoApply` on the candidate policy. | No drive-map content/link proof, client `gpresult`/RSoP, root cause, or mapped-drive validation. |
| [HD-005](../tickets/HD-005-vpn-mfa.md) | **Illustrative scenario** | Wrote a provider-neutral VPN/MFA diagnostic and communication model. | Ticket narrative only; the osTicket screenshots show seeded case-study records, not execution. | Reasoning artifact only. | Requires correlated live VPN, identity-provider, and MFA logs plus a tested connection. |
| [HD-006](../tickets/HD-006-slow-pc-malware-escalation.md) | **Illustrative scenario** | Wrote a policy-aware triage, containment, preservation, and escalation model. | Ticket narrative only; the osTicket screenshots show seeded case-study records, not execution. | Reasoning artifact only. | Requires endpoint telemetry, recorded containment/handoff, and validated remediation or release. |
| [HD-007](../tickets/HD-007-phishing-escalation.md) | **Illustrative scenario** | Wrote an evidence-preserving phishing-report and escalation model. | Ticket narrative only; the osTicket screenshots show seeded case-study records, not execution. | Reasoning artifact only. | Requires the original message, headers/content, tenant telemetry, and recorded response actions. |

## Interpretation rules

- AD/GPO console output proves only the visible directory or configuration state at capture time.
- A seeded ticket narrative is not evidence that the described support or security workflow ran.
- **Exercise outcome** in each ticket is separate from its captured osTicket workflow state.
- Hashes, dimensions, artifact-specific claims, edit/redaction notes, and limitations are in the
  complete [evidence manifest](../evidence/README.md).
