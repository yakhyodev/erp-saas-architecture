# ADR-INT-005 — Contragent/SalePoint Normalization

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
External systems may distinguish payer, counterparty and outlet differently.

## AR-008 contract proposal
Transactions should reference explicit roles:
- payer;
- counterparty;
- outlet/sales point.

Canonical ownership is unresolved. This ADR does not approve a CRM/Finance model.

## Variantlar
Flatten identities; mirror provider mode; separate roles.

## PROPOSED yechim
Preserve roles and source mappings while awaiting CRM + Finance approval.

## Status
OPEN dependency on AR-008.
