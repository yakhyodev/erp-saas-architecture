# CRM_DISTRIBUTION.md — CRM & Distribution Architecture

- **Version:** 0.2 (architecture consolidation, 2026-10-11)
- **Domain owner:** CRM & Distribution
- **Status:** PROPOSED for all new module architecture, separate from APPROVED platform constraints
- **Scope:** Customer, Counterparty, Outlet, Field Agent, Supervisor, Territory, Route, Visit, GPS, photo evidence, Distribution, KPI and SalesDoc adapter.
- **Implementation:** no application code.

## 1. APPROVED platform requirements (inherited; not CRM decisions)

Hybrid Prepaid SaaS = inexpensive Seat + Business Usage (base charge per document and additional credits beyond volume threshold) + Resource Usage; **1 credit = 10 UZS**. Progressive usage discounts and enterprise individual agreements. When the balance is exhausted, **actual** seat and usage costs accrue as a negative balance for **one calendar month**, without estimating a credit limit. A unified SaaS platform handles companies and their branches. UI: Uzbek, English, Russian plus hidden future Uzbek alphabet option.

Go, PostgreSQL, Next.js and Flutter remain **recommended, NOT APPROVED**. Modular monolith and multi-tenant architecture are under review; multi-server, sharding and enterprise isolation are future considerations.

## 2. Bounded contexts and ownership (PROPOSED)

| Context | Owns | Outside its authority |
|---|---|---|
| CRM | Counterparty profile / customer relationship, outlet, contacts, dedupe lineage | AR ledger, sales orders, staff login |
| Field Force | Operational agent profile, supervisor association | Global identity and HR payroll |
| Territory | Hierarchy, geographic/list-based coverage, effective-dated assignments | Tenant memberships |
| Route / Visit | Plan, stop, check-in/out, outcome, evidence references | Product stock and sales order posting |
| Distribution | Delivery assignment, expeditor dispatch, proof/exception coordination | Financial posting and inventory movement |
| Performance | KPI rule versions, results, bonus proposals | Payroll execution |
| SalesDoc Adapter | External ID mapping, sync cursors, reconciliation | Canonical ERP domain ownership |

Customer is a commercial relation to a Counterparty, not a second disconnected legal identity. **Open conflict:** whether Counterparty master ultimately belongs to CRM or a shared Party service serving Purchasing/Finance. Validate tenant **and** branch/resource scope for every read, write and cross-domain reference.

## 3. Logical entities (PROPOSED, not SQL)

- **Counterparty:** immutable opaque ID, person/organization, legal name, jurisdiction-specific identifiers, contacts, status, audit, tenant-scoped merge/dedupe lineage.
- **Customer:** Counterparty ID, relationship state, segment, account owner; no mutable ledger balance.
- **SalesOutlet:** customer ID, physical/virtual point, code, address, optional coordinates, contact, status; one customer can have many outlets.
- **Contact:** contact channels, party/outlet link, consent/preference and PII controls.
- **FieldAgent:** Identity/HR reference, role/team/branch eligibility, active dates.
- **SupervisorAssignment:** supervisor-to-agent relationship with effective dates and audit.
- **Territory:** named hierarchy, optional geometry/polygon, lifecycle and effective date; geo polygons not required at MVP.
- **TerritoryAssignment:** territory/agent/team/outlet association, effective dates, overlap/conflict controls.
- **RoutePlan / RouteStop:** date and business timezone, agent, version/state; ordered stops, outlet and planned visit windows.
- **Visit:** agent/outlet, route reference optional, workflow state, purpose, check-in/out and server/device timestamps.
- **LocationEvidence:** purpose, coordinates only if authorized, accuracy/source, device time/server receipt, integrity flags.
- **PhotoReport:** visit reference, type, private object key, checksum, captured/uploaded times, retention/moderation.
- **DeliveryAssignment:** authoritative Sales fulfillment reference, expeditor, route, delivery window, operational state.
- **ProofOfDelivery:** delivery reference, accepted quantity, recipient evidence, exceptions, timestamps and sync status.
- **KPIDefinition:** versioned formula metadata, targets, source, applicability, period/timezone/effective dates.
- **KPIResult / BonusProposal:** rule version, source lineage, amount/score, review/approval, corrections; not payroll/finance posting.
- **ExternalMapping / SyncJob:** tenant-scoped vendor ID to canonical ID, sync cursor, retry, errors, mapping version.

Every mutable entity has tenant scope, version, lifecycle status and timestamps; branch access is separately authorized. Preserve historical assignments; don't silently delete transaction-referenced parties.

## 4. Main workflows (PROPOSED)

**Customer onboarding:** search duplicates → validate counterparty/contact → create customer/outlet → approve if policy requires → effective-dated agent/territory assignment → publish changed-entity event. Merge needs permission, review and lineage.

**Agent/supervisor:** Identity owns authentication; CRM stores only operational profiles and effective-dated roles/teams/coverage. Reassignment must preserve prior sales/visit/KPI attribution. A supervisor only views permitted team data.

**Territory/route:** area may be customer-list-based or geographic. Route suggested states: DRAFT → PUBLISHED → IN_PROGRESS → COMPLETED, with CANCELLED/REVISED branches and audit. Manual order first; route optimization optional and not approved. Publish validates branch, agent, outlet, schedule and conflicts. Revision to an offline device requires explicit conflict handling.

**Visit/GPS:** suggested PLANNED or AD_HOC → CHECKED_IN → CHECKED_OUT → SUBMITTED → VERIFIED, plus REJECTED/NEEDS_REVIEW/CANCELLED. GPS capture only for lawful, disclosed work purposes; default excludes off-duty continuous tracking. Differentiate missing GPS, inaccurate GPS, out-of-radius and suspected spoofing—these are not automatic fraud determinations. No geofence distance, accuracy threshold, legal basis, retention schedule or required-photo policy is yet approved.

**Mobile offline:** propose encrypted device queue, generated idempotent command IDs, ordered visit mutation sync, server transition and permission revalidation, visible pending/conflict state and private resumable media uploads. Do not let offline acceptance bypass Sales credit/stock decisions.

**Distribution:** Sales creates fulfillment-ready request → dispatcher assigns expeditor / optional delivery route → delivery evidence/exception submitted → Sales confirms quantities, returns and stock → Finance handles authorized collections/AR. Operational states suggested: READY → ASSIGNED → IN_TRANSIT → DELIVERED, with PARTIAL/FAILED/RETURN_REQUESTED/CANCELLED. Fleet management and cash-on-delivery module boundary are open.

**KPI/bonus:** potential metrics include planned/productive visits, coverage, verified visits, net accepted sales, new active outlets, successful deliveries, returns and collection quality. Formulas, targets, exclusions and payout rates remain OPEN. Preserve KPI policy version, source event lineage, provisional vs finalized periods. Bonus proposal → review → approval → HR/Finance handoff → payment owner confirmation. Corrections use append-only adjustments, not silent overwrites.

## 5. Inter-module data flows (PROPOSED)

| From → To | Information |
|---|---|
| Tenant / Identity → CRM | tenant membership, branch access and actor permissions |
| CRM → Sales / Distribution | CustomerChanged, OutletChanged with canonical IDs |
| Visit → Sales | CreateSalesOrder request with idempotency key; Sales owns result |
| Sales → Distribution | FulfillmentReady, OrderChanged, DeliveryCancelled |
| Distribution → Sales | DeliveryEvidenceSubmitted, DeliveryExceptionReported |
| Sales / Finance → KPI | authoritative net sales, returns, AR/collection snapshots |
| Visit / Distribution → KPI | verified visits, routes and delivery outcomes |
| KPI → HR / Finance | approved bonus proposal / adjustment for acceptance |
| CRM / Distribution → SaaS Billing | deduplicated *candidate* metering event; Billing owns rate, wallet, discount and negative-balance rules |
| External SalesDoc ↔ Integration | mapped IDs, import/export, cursor, errors, reconciliation |

Recommend tenant-scoped outbox/inbox, event schema version, event ID, aggregate version, occurred-at UTC, correlation ID, actor and minimal payload. Never broadcast raw GPS, photos or tax IDs. Support retries/reconciliation without duplicate business documents or duplicate SaaS charges.

## 6. Security, privacy, scale (PROPOSED)

Enforce role + tenant + branch + assignment + purpose across APIs, offline sync, exports, evidence and events. Encrypt device cache, transport and private object data; signed time-limited media access, checksum, malware/EXIF handling, audit high-risk access. Collect minimal GPS only while authorized; implement legally required notice, retention and access/delete controls. Retention, media quotas and exact policies remain OPEN.

Prefer module boundaries compatible with a modular monolith if platform approves. Scalable sync/media/integration/KPI workers; no local-only persistent state. Index access by tenant/branch/outlet/agent/status/time; Database team owns storage decisions. Store photo bytes outside transactional rows; location precision and archival policy require review. Monitor event lag, sync conflicts, route completion, uploads, delivery exceptions and KPI recomputation without leaking sensitive coordinates.

## 7. Architectural Decision Records (all PROPOSED)

- [ADR-CRM-001 — Domain ownership](../adr/ADR-CRM-001-domain-ownership.md)
- [ADR-CRM-002 — Offline visits, GPS and privacy](../adr/ADR-CRM-002-offline-gps-privacy.md)
- [ADR-CRM-003 — SalesDoc anti-corruption adapter](../adr/ADR-CRM-003-salesdoc-adapter.md)
- [ADR-CRM-004 — Effective-dated assignments](../adr/ADR-CRM-004-effective-dated-assignments.md)
- [ADR-CRM-005 — KPI and bonus separation](../adr/ADR-CRM-005-kpi-bonus-separation.md)

## 8. OPEN QUESTIONS

1. Counterparty canonical master owner (CRM vs common Party) with Finance/Purchasing?
2. Overlapping outlet assignments to branches, agents, territories; precedence?
3. Counterparty dedupe/merge, approval and cross-branch visibility policy?
4. Route publish/reorder authority and offline revision conflicts?
5. Visit verification, geofence accuracy/radius, GPS lawful basis/retention and work hours?
6. Delivery vs standalone Logistics boundary; fleet, cash on delivery and returns?
7. KPI formulas, bonus eligibility, reversals, approvers and period locking?
8. SalesDoc API auth, supported endpoints, rate limits, versions and data ownership—**unverified**.
9. Usage metering taxonomy, photo quotas, privacy/SLO/performance limits and corporate timezone?

## 9. Shared documentation changes — SUGGESTED ONLY, NOT APPLIED

After approval, register ADRs in `DECISIONS.md`, domain ownership in `MASTER_ARCHITECTURE.md`, accepted functional/privacy rules in `REQUIREMENTS.md`, interfaces in `MODULE_REGISTRY.md`, unknowns in `OPEN_QUESTIONS.md` and dated changes in `CHANGELOG.md`. Do not overwrite other chats' documentation.

## 10. CODEX SPECIFICATION — FUTURE IMPLEMENTATION ONLY

No code now. After explicit approvals Codex should deliver:

1. Reviewed entities, state transitions and versioned contracts; stable IDs and tenant/branch-aware authorization.
2. CRUD/command/query interfaces for party/outlet, agent assignments, territories, routes, visits, photo/GPS evidence, dispatch/POD, KPI/bonus approvals, sync/reconciliation; cursor pagination and field permissions.
3. Reliable outbox/inbox event delivery with retries, idempotency, dead-letter/quarantine, audit and correlation.
4. Encrypted, revocable mobile offline cache and sync queue; conflicting revisions go to review, not silent acceptance.
5. Protected geospatial/media storage, minimum collection, limited-time object access, retention and redaction.
6. Versioned KPI engine, period snapshots, reversible adjustments, authority-respecting HR/Finance bonus handoff.
7. SalesDoc adapter only after verified API contract, with external ID mappings, backfill/replay and reconciliation.
8. Cross-module contract tests and load/security/recovery tests. Block cross-tenant/branch access; no duplicate visits/orders/POD/usage charges; never directly post ledger or stock from CRM; historical attribution remains immutable.

**Definition of Ready:** ADR approvals, Party owner, access/branch model, route/visit/offline/GPS policy, Delivery vs Logistics owner, KPI/bonus rules, SalesDoc contract and Billing metering categories. This specification is NOT authorization to implement.
