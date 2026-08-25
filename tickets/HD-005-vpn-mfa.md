# HD‑005 — VPN won't connect (fails at MFA step)

> **ILLUSTRATIVE SUPPORT SCENARIO — NOT A LIVE VENDOR OR SYSTEM RESULT.** The narrative below
> demonstrates a safe diagnostic approach. No VPN, identity-provider, MFA, or device logs are
> included, so no root cause or successful connection is evidenced.

| | |
|---|---|
| **Ticket #** | 653087 |
| **Reported by** | Tom Becker (tbecker), Sales |
| **Help topic** | Network & Remote Access |
| **Priority** | High |
| **Classification** | Illustrative support scenario |
| **Exercise outcome** | Diagnostic example only; no live outcome evidenced |
| **Captured osTicket state** | Open / Unassigned |

## Summary

A remote user reports that VPN authentication reaches the MFA stage and then displays an
"authentication timeout."

> *"Working from home and the VPN accepts my password but then fails on the MFA step with an
> authentication timeout. Nothing changed on my end." — Tom Becker, Sales*

## Impact & priority

If current, a fully remote user unable to reach internal resources has **High** individual impact.
Confirm scope before concluding that only one user is affected.

## Provider-neutral, log-driven investigation

A UI message is not proof that the password was correct, that the account is healthy, or that a
specific MFA factor failed. Correlate the attempt by timestamp, username, request/correlation ID,
and source IP across:

1. VPN gateway or client logs: transport, tunnel, and authentication-stage result.
2. Identity-provider sign-in logs: first-factor result, conditional-access/policy decision, and
   failure code.
3. MFA-provider logs: factor offered, challenge delivery/validation, denial/timeout, and device or
   enrollment state.
4. Service health and scope: one user versus multiple users, recent policy/configuration changes,
   and provider incidents.

Then verify enrollment and the intended factor. For TOTP, confirm automatic device time and try a
fresh code; for push, confirm connectivity and that the user initiated the request; for a replaced
device, follow approved identity verification and re-enrollment procedures. Never approve an
unexpected prompt.

## Illustrative hypotheses, not findings

Clock drift, push delivery failure, stale enrollment, conditional-access policy, account state,
VPN/RADIUS integration, or a provider outage could all produce similar symptoms. Without the
correlated logs, this case cannot identify one as the root cause and cannot claim a successful
post-change VPN connection.

## Communication template

> Hi Tom, we have the reported MFA timeout. Please send the approximate attempt time and any
> correlation/error code shown. Do not approve unexpected prompts. We will correlate the VPN,
> identity, and MFA logs before recommending a provider-specific step. — IT Help Desk

## Escalation decision

Escalate multiple-user failures or service-health indicators to the VPN/identity owner. Treat
unrequested repeated prompts, impossible travel, unfamiliar source IPs, or other suspicious
sign-ins as a security event.

## Follow‑up

Use **[KB‑003 — VPN + MFA troubleshooting](../knowledge-base/KB-003-vpn-mfa-troubleshooting.md)**
for the evidence-driven procedure.
