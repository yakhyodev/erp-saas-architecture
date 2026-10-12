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

## Round 2 decision packet — MR-13 (PROPOSED; 2026-10-12)

**Problem:** GPS/photos are sensitive field-worker information and may be unavailable offline. **Options:** visit-only collection, explicitly authorized on-duty route sampling, continuous monitoring; consent-only or jurisdiction-appropriate other lawful basis; blanket TTL or purpose/regional retention; hard GPS rejection or human-reviewed exception. **Recommendation:** purpose-limited check-in/out evidence, no off-duty default tracking; capture source/accuracy/device/server time and missing/low-accuracy/spoof statuses; encrypted device queue and private media refs with EXIF minimization. Review denied/missing evidence rather than inventing coordinates or automatically accusing agents. Consent/lawful basis must be established separately per jurisdiction.

**OPEN user/legal decisions:** lawful basis/consent/withdrawal, tracking hours, accuracy/geofence thresholds, mandatory photos, retention by purpose and jurisdiction, erasure/legal holds, offline TTL, evidence access and appeal. **Safety boundary:** no raw GPS/photos in domain events/KPI logs; device revocation triggers server-side rejection of replay; remote wipe while offline not guaranteed. **Impact:** Mobile, Security, UX-T09/T11, Distribution, Infra, Localization, legal/privacy. **Acceptance specification, not tests executed:** revoked sync rejected; missing GPS distinguished; evidence private; retained evidence policy-guarded. Not approved; no code.
