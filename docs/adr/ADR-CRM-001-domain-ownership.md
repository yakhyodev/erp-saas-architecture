# ADR-CRM-001 — Domain ownership

- **Status:** PROPOSED
- **Date:** 2026-10-11

## Problem
CRM, Sales, Finance, Tenant and Billing may duplicate party, order, receivable and delivery authority.

## Options
All-in-one data ownership; modular bounded contexts; immediate microservices.

## Proposed decision
CRM owns customer relations, outlets, route/visit evidence and agent assignments. Canonical Counterparty/Party ownership remains **OPEN** between CRM and a separate shared logical Party context, to be aligned with Finance/Purchasing. **Reconciled PROPOSED domain map:** a Catalog boundary owns canonical Product/SKU/UoM; Pricing owns PriceList; Sales owns order/commercial document lifecycle and fulfillment orchestration; Inventory owns stock, reservation and movement ledger; Purchasing owns purchase documents; Finance owns receivables, ERP GL and payments; Tenant manages scoped membership lifecycle, Identity credentials, Security roles and permission evaluation (final mapping OPEN); Billing owns SaaS wallet, seat and usage rating. Interactions use immutable canonical IDs, verified tenant scope, and versioned commands/events. Historical draft wording grouped Product and Inventory under Sales; this is superseded as a **proposal only** by the separate-domain recommendation above.

## Rationale
Avoid inconsistent masters while supporting modular-monolith deployment.

## Risks
Eventual consistency, duplicate party merges and stale read models.

## Other modules affected
Master Architecture, Database, Sales, Finance, Tenant, Billing and Integration.

**Approval pending. No code authorized.**
