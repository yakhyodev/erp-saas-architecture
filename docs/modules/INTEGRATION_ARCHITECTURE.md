# INTEGRATION_ARCHITECTURE.md

- Version: 0.2 — 2026-10-11
- Status: **PROPOSED** except explicitly identified approved business rules
- Scope: External integrations, SalesDoc compatibility, migration contracts, API/event evidence.

## Evidence taxonomy (AR-011) — PROPOSED

External integration claims MUST use one of these labels:

| Level | Meaning |
|---|---|
| DOCUMENTED | Exists in published API/schema/documentation. Does not prove production behavior. |
| LIVE_VERIFIED | Verified against an authorized live environment with recorded evidence. Not achieved for SalesDoc in this architecture phase. |
| INFERRED | Reasoned from documentation or domain behavior; requires validation before contract use. |
| UNVERIFIED | Unknown; must not be implemented as a guaranteed capability. |

Current SalesDoc analysis remains DOCUMENTED only. No production API, credentials, sandbox, or write operations were used.

## Canonical integration boundary — PROPOSED

External systems connect through a versioned adapter boundary. External contracts are not ERP domain contracts.

Provider Adapter → Normalization → External Identity Mapping → Staging → Domain-owned Import/Command Contract → Reconciliation

Each adapter contract MUST define:
- provider name and contract version;
- supported methods/events;
- evidence level per capability;
- identity mapping rules;
- ownership of imported facts;
- retry/idempotency rules;
- cutover and rollback evidence requirements.

## AR-008 payer/counterparty/outlet identity mapping — PROPOSED

Open question remains: canonical owner model is not approved.

Candidate models:

1. Single Party registry with roles:
   - Party
   - Counterparty role
   - Payer role
   - Outlet/sales point role

2. Bounded context ownership:
   - CRM owns relationship identity;
   - Finance owns payer/debt identity;
   - Sales owns commercial participant references.

Current proposal:
- never derive payer ownership only from SalesDoc server mode;
- preserve external source relationships in mapping records;
- transactions reference explicit roles, not ambiguous client IDs.

Decision requires CRM + Finance + Sales approval.

## AR-010 imported external document charge lifecycle — PROPOSED

Imported documents MUST have separate lifecycle from native ERP documents.

Proposed states:

DISCOVERED → STAGED → VALIDATED → MAPPED → ACCEPTED → POSTED → RECONCILED → ARCHIVED

Rules:
- imported payload hash prevents duplicate processing;
- external ID + provider + tenant + entity type is not sufficient alone if source revisions exist;
- corrections create reconciliation events, not silent overwrite;
- chargeability is unresolved and belongs to Billing + Sales/Inventory.

No decision is made that imported documents are automatically chargeable.

## MR-10 versioned API/event contracts — PROPOSED

Integration contracts require:

- immutable contract version;
- compatibility policy;
- migration path between versions;
- event schema ownership;
- backward compatibility window;
- deprecation record.

Events are preferred for durable synchronization, but no event technology choice is approved.

## Migration cutover and rollback evidence — PROPOSED

Before cutover:
- source snapshot/checkpoint recorded;
- imported counts reconciled;
- financial/inventory owners sign acceptance;
- rollback point documented.

Rollback MUST not create duplicate documents or reverse financial facts without domain approval.

## Approved business rules preserved

No changes to:
- Hybrid Prepaid SaaS;
- Seat + Business Usage + Resource Usage;
- 1 credit = 10 so'm;
- progressive usage discounts and enterprise contracts;
- real Seat/Usage monthly negative balance behavior;
- unified company/branch SaaS;
- uz/en/ru and hidden future Uzbek alphabet option.

## Existing ADR status

ADR-INT-001..005 remain **PROPOSED** only. No approval transition performed.
