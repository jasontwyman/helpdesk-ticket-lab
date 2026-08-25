# KB‑003 — VPN + MFA connection troubleshooting

**Applies to:** Remote-access VPN with multi-factor authentication

**Audience:** Tier‑1 help desk

**Related ticket:** [HD‑005](../tickets/HD-005-vpn-mfa.md)

## Evidence rule

User-facing wording such as "password accepted" or "MFA timeout" identifies where the UI appeared
to fail; it does not prove account health, password correctness, factor validation, or root cause.
Use correlated logs and provider documentation before selecting a fix.

## Record the attempt

Capture username, timestamp/time zone, source IP/network, VPN client/gateway, exact error and code,
request/correlation ID, MFA method, device change, and whether the user initiated the attempt.
Confirm whether one or multiple users are affected and check service health/change records.

## Correlate the authentication chain

1. **VPN client/gateway logs:** connectivity, tunnel stage, authentication broker/RADIUS result,
   and returned error.
2. **Identity-provider sign-in logs:** account state, first-factor result, policy/conditional-access
   decision, risk signals, and failure code.
3. **MFA-provider logs:** challenge offered, delivery/validation status, denial/timeout, enrollment,
   and device/factor state.
4. **Post-connect logs:** if authentication succeeded, inspect tunnel, routing, DNS, and
   authorization separately.

Document which system produced each conclusion; do not infer a successful first factor from the
VPN UI alone.

## Provider-neutral checks by factor

* **TOTP/code:** verify the device uses automatic date/time and time zone, then try a newly
  generated code. Use the organization's/provider's documented resynchronization flow if logs
  indicate code-window mismatch; menu paths vary by app and version.
* **Push:** verify the user initiated the request, phone connectivity, app notifications, and the
  challenge shown. Deny unexpected prompts—never coach repeated blind approvals.
* **SMS/voice:** verify the registered destination and provider delivery status without exposing
  the full number. Follow policy for fallback factors.
* **New/replaced device:** re-enroll only after approved identity verification; do not bypass MFA.
* **Policy/account:** resolve lockout, disablement, risk policy, factor restriction, or licensing
  only when the authoritative logs show it.

## When to escalate

* Multiple users, provider health alarms, or a common failure code → VPN/identity owner.
* Repeated unsolicited prompts, unfamiliar sources, risky sign-ins, or suspected push bombing →
  Security; tell the user to deny and report.
* Missing/inaccessible logs or contradictory results → the owning technical team.

## Closure evidence

Record the change made, correlated log outcome, and a successful controlled reconnection if one
occurred. Without those artifacts, mark the ticket diagnostic/pending rather than resolved.
