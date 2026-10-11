# ADR-CRM-001 — Domain ownership

- **Status:** PROPOSED
- **Date:** 2026-10-11

## Problem
CRM, Sales, Finance, Tenant and Billing may duplicate party, order, receivable and delivery authority.

## Options
All-in-one data ownership; modular bounded contexts; immediate microservices.

## Proposed decision
CRM owns customer relations and outlets; canonical Counterparty ownership still needs alignment with Finance and Purchasing. Sales owns products, orders, inventory and fulfillment; Finance owns receivables, ledger and payments; Tenant/Identity owns membership and credentials; Billing owns wallet, seat and usage prices. Interactions use canonical IDs, tenant checks and explicit command/event contracts.

## Rationale
Avoid inconsistent masters while supporting modular-monolith deployment.

## Risks
Eventual consistency, duplicate party merges and stale read models.

## Other modules affected
Master Architecture, Database, Sales, Finance, Tenant, Billing and Integration.

**Approval pending. No code authorized.**
