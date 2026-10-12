# ADR-MOB-009 — Mobile Authentication and Device Security

- **Status:** PROPOSED
- **Owner:** 09 Mobile & Offline
- **Review:** AR-006 / AR-013
- **Date:** 2026-10-11

## Problem
Mobile devices store scoped ERP data while offline. Sessions may expire or permissions may be revoked before queued commands reach the server. An offline device cannot reliably receive revocation immediately.

## Alternatives
1. Persistent bearer token and indefinite offline access: rejected due to credential risk.
2. Online-only authentication: incompatible with offline workflows.
3. Short-lived access, protected refresh and bounded offline session with server rechecks: **PROPOSED**.

## Proposed contract
- Access token short-lived; refresh token stored using OS-protected secret storage and rotated according to Security-owned policy. Do not store raw tokens in ordinary local database or logs.
- Local database/media encrypted; encryption keys bound to OS secure storage where supported; tenant/account/device data isolated. Device registration and revocation are server-managed.
- Offline access is bounded by a Security-approved duration and role/scope policy. Local offline authentication is a UX gate, **not** permission to commit server transactions.
- On reconnect, re-authenticate/refresh as appropriate. Before processing **every** queued command, server rechecks current identity, membership, branch assignment, device status and operation permission. Expired/revoked credentials or removed membership lead to rejection or reauthentication, never automatic approval.
- Revocation while offline cannot guarantee immediate remote wipe. On detection, block restricted operations and follow Security-approved quarantine/purge procedure; preserve evidence only as permitted by retention/audit rules.
- Tenant switch/logout must not expose the previous tenant's cached records or encryption keys. Pending operations must be resolved under explicit policy, not silently sent as another account.
- Token refresh errors must not create infinite retry loops or leak sensitive data. GPS/media upload authorization is checked independently at server receipt.

## Rationale
Balances offline usability with least privilege and authoritative security controls.

## Risks
Device theft, delayed revocation, OS background limitations, key loss, unprocessed outbox, and sensitive evidence retention.

## Cross-module impact
Security & Permissions and Multi-Tenant own identity, membership and revocation policy; Backend validates every command; Mobile implements storage/session UX. No unilateral approval of TTL, token lifetime, remote wipe guarantees or data retention.

## Open questions
Offline session maximum; token rotation and device trust; revocation cleanup; account switch with pending queue; biometric/local PIN requirement; GPS/media retention. All **OPEN**.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md) · [Open Questions](../../OPEN_QUESTIONS.md)

## Round 2 refinement — AR-006 / MR-12 (PROPOSED)

**This ADR already exists.** No status promotion; Security remains owner of authentication, revocation, TTL, device trust and retention policy.

### Reconnect validation sequence

1. Validate credential signature/session/expiry and current server revocation state. Expired access token requires authorized refresh/reauth; revoked device cannot refresh.
2. Resolve authoritative identity, active tenant membership, company and branch entitlements, role grants (including revoked roles), resource assignment and target operation permission.
3. Validate registered device/session family and bound tenant context; deny cross-tenant replays even when cached UI previously permitted them.
4. Apply business validation and idempotency only within verified scope; audit denied attempts. Repeat checks for **each** queued command, GPS/media upload and asynchronous processing step as relevant.

### Local data and device policy options — OPEN

| Topic | PROPOSED alternatives | Non-negotiable boundary |
|---|---|---|
| Offline session | bounded offline grant, stricter sensitive-role grant, online-only high-risk action | no indefinite implicit server authority |
| Device revoke | stop refresh/server sync; detect and lock local access on next contact | cannot promise immediate remote wipe offline |
| Encrypted local data | OS-keystore backed per-account/scope keys; crypto erase on verified revoke | never expose plaintext on tenant switch |
| Pending evidence | quarantine encrypted for authorized review; purge after approved retention; legal hold exception | no unauthorized upload or silent loss |
| Cleanup | acknowledgement-based media cleanup; policy-bound deletion; documented recovery/export | no unilateral retention period |
| Theft / >30 days offline | local lock after policy-defined TTL; server rejects revoked/expired rights | 30+ days is a **test scenario**, not a TTL |

Token values and encryption keys never enter application logs. Logout/tenant switch partitions or locks old-scoped pending data; never resubmit under a different account. Permission revocation does not by itself determine legal evidence deletion. Define cleanup/retention jointly with Security, CRM and applicable policy owners.

**OPEN:** precise offline TTL, refresh lifetime, biometric/PIN gate, device attestation, device revocation recovery, legal hold, evidence retention, lost-key recovery and cross-tenant pending draft treatment. All require Security/domain sign-off.
