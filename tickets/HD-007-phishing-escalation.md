# HD‑007 — Suspicious email asking me to verify my password

> **ILLUSTRATIVE SECURITY SCENARIO — NOT A LIVE MAIL-TENANT RESULT.** No original message,
> headers, body, URLs, attachments, tenant telemetry, purge record, or blocking action is included
> in this repository. The workflow below is guidance, not evidence that Security performed it.

| | |
|---|---|
| **Ticket #** | 700501 |
| **Reported by** | Omar Hassan (ohassan), Finance |
| **Help topic** | Security Incident |
| **Priority** | High |
| **Classification** | Illustrative support scenario |
| **Exercise outcome** | Escalation example only; no live outcome evidenced |
| **Captured osTicket state** | Open / Unassigned |

## Summary

A user reports an email claiming to be from IT that asks him to click a link and enter his
password. He reports that he did not click.

> *"I got an email claiming to be from IT asking me to verify my account by clicking a link and
> entering my password. Looks off – I did NOT click it. Reporting it to be safe." — Omar Hassan,
> Finance*

## Impact & priority

Credential-phishing indicators justify **High** priority and prompt Security review. One report
may or may not represent a broader campaign; tenant telemetry is required to establish scope.

## Preserve complete evidence without interacting

For a live case, use the approved report-phishing mechanism or preserve the original `.eml`/`.msg`
and collect:

* full transport and authentication headers;
* subject, timestamps, sender/envelope sender, recipients, and the complete message body;
* displayed link text and normalized/defanged URL destinations obtained safely from message
  source or security tooling, not by browsing to them;
* attachment names, types, sizes, and hashes/analysis references when available through approved
  tooling—do not open them on a user endpoint;
* user interaction details (opened, clicked, downloaded, executed, credentials entered, MFA
  approved), with approximate times; and
* tenant telemetry such as message trace, campaign/cluster identifiers, delivery locations,
  related recipients, click telemetry, mail-security verdicts, and post-delivery actions.

Headers are important but are **not the entire investigative value**; body content, URLs,
attachments, user actions, and tenant telemetry are also necessary.

## Security handoff and proportional response

Security should validate indicators and scope before selecting controls. Potential actions include
message search and purge, URL/file-indicator blocking, account/session containment for affected
users, endpoint review, and campaign notification. Block the narrowest validated indicators.
**Do not blanket-block an entire sender domain** solely because one address was spoofed or one
message linked to a compromised legitimate site; broad domain blocks can cause substantial
collateral damage.

If credentials were entered or an unexpected MFA prompt was approved, escalate immediately for
account disable/reset as policy requires, token/session revocation, sign-in review, mailbox-rule
review, and endpoint/tenant investigation.

## Communication template

> Hi Omar, thank you for reporting this and for not interacting with it. Please keep the original
> message available and use the approved Report Phishing action if you have not already. Security
> will validate the message and tenant telemetry; we will contact you if additional containment is
> required. — IT Help Desk

## Evidence boundary

The user's quoted report is scenario context. This case does not prove that the message was
phishing, that other recipients existed, or that Security blocked, purged, or investigated it.
Those results must come from the live mail/security systems.

## Follow‑up

Use **[KB‑004 — Reporting and handling a suspected phishing email](../knowledge-base/KB-004-phishing-response-runbook.md)**.
