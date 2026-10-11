# ADR-CRM-002 — Offline visits, GPS and privacy

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** CRM & Distribution
- **Related module:** [CRM_DISTRIBUTION.md](../modules/CRM_DISTRIBUTION.md)

## Problem
Unreliable mobile connectivity and sensitive, high-volume evidence.

## Options considered
Online-only capture; offline-first purpose-scoped capture; continuous tracking by default.

## Proposed decision
Encrypted offline queue with server validation, idempotency, scoped check-in/out location and private photo evidence; no default off-duty/continuous tracking.

## Rationale
Field usability, resilient synchronization, data minimization.

## Risks
GPS spoofing, device compromise, concurrency, legal requirements.

## Impact on other modules
Mobile/Offline, Security, Infrastructure, Database, Billing, UX.

## Approval
Not approved. Requires cross-module architecture review. No application code authorized.
