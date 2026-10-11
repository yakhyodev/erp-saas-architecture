# ADR-INT-002 — Scoped External Identity Registry

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
CS_id, SD_id and code_1C cannot be assumed globally unique.

## Proposed contract
External identity key scope:
Tenant + Provider + Connection + Source Server + Filial + Entity Type + Identifier Type + Identifier Value.

Conflicts MUST quarantine. Silent merge is forbidden.

## AR-008 relation
Payer, counterparty and outlet identity mapping remains OPEN. Identity registry stores mappings but does not decide business ownership.

## Variantlar
External IDs as primary keys; source composite keys; immutable ERP IDs with mapping.

## PROPOSED yechim
Immutable ERP IDs plus scoped external identity mapping.

## Status
Requires Database, Multi-Tenant, CRM and Finance review.
