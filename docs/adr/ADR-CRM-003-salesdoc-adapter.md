# ADR-CRM-003 — SalesDoc anti-corruption adapter

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** CRM & Distribution
- **Related module:** [CRM_DISTRIBUTION.md](../modules/CRM_DISTRIBUTION.md)

## Problem
External schemas, IDs and workflows differ from canonical ERP.

## Options considered
Direct coupling; versioned adapter; manual/CSV-only migration.

## Proposed decision
Adapter with versioned mapping, tenant-scoped external IDs, cursor, inbox/outbox and reconciliation. API contract not yet verified.

## Rationale
Reduce vendor lock-in and support safe incremental migration.

## Risks
Changes, rate limits, incorrect mappings, unknown API permissions.

## Impact on other modules
Integration, Security, CRM, Sales, Billing, Analytics.

## Approval
Not approved. Requires cross-module architecture review. No application code authorized.

## Round 2 decision packet — AR-008/AR-011 (PROPOSED; 2026-10-12)

**Problem:** SalesDoc external identifiers and vendor claims must not be treated as canonical Party identities or production proof. **Alternatives:** direct CRM provider IDs, source-scoped adapter mapping (**recommended**), manual reconciliation only. **Recommended contract:** unique scoped key (tenant, provider, connection, external type, external ID); mapping to stable internal Party/Customer/Outlet IDs, versioned mappings, alias lineage, conflict quarantine, source occurrence dedup and auditable merge/reassignment. Finance must validate posted legal debtor/payment obligations independently.

**Evidence:** DOCUMENTED = public provider documentation with source/revision/access date; LIVE_VERIFIED = endpoint/environment/time/test result and sanitized evidence from authenticated call; INFERRED = analysis only; UNVERIFIED = no sufficient proof. Documented never implies live verified. **No SalesDoc endpoint or production authorization was live-tested by this ADR.**

**Risks:** provider ID collisions/reuse, tenant leakage, stale mappings, imported-doc double charges, unverified API assumptions. **Impact:** Integration evidence ledger and adapters, CRM/Party, Sales documents, Finance debtor mapping, Billing usage classification, Security. **OPEN:** API scopes/version/rate limits, historical import billing trigger, cutover and rollback evidence, Party writer approval. No code.
