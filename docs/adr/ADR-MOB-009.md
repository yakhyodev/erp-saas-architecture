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
