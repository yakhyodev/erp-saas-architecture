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

## Round 2 decision packet — AR-002/AR-008, MR-09 (PROPOSED; 2026-10-12)

**Problem:** CRM-owned Counterparty vs shared logical Party and accidental assumption that Sales is Product/Pricing/Inventory master. **Options:** A shared logical Party context with one writer (**recommended**, initially inside modular monolith); B CRM-owned Party identity writer; C multiple masters with reconciliation (not recommended). **Decision:** none approved; Party writer and legal-identity merge are **OPEN**. CRM remains sole writer of customer relationship and outlet data; Purchasing owns supplier role; Finance owns payer settlement, AR legal debtor snapshots, GL; Sales owns orders; Catalog owns SKU/Product/UoM; Pricing owns price rules; Inventory owns reservations/stock movements. A shared context is a logical boundary, not approved microservice.

**Reason:** a sole canonical ID authority avoids divergent legal identities; independent roles preserve financial obligations. **Risks:** extra boundary, mistaken party merge, cross-company identity leakage. **Required contracts:** tenant-qualified party ID, effective-dated role link, verified scope, versioned PartyCreated/PartyMerged/PartySplit with audit and alias lineage; Finance posted debtor snapshots remain immutable. **Impact:** CRM, Catalog, Pricing, Sales, Inventory, Purchasing, Finance, Tenant/Security, Integration, UX. Approval is pending; no code.
