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
