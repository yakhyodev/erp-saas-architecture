# ADR-INT-002 — Scoped External Identity Registry

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
CS_id, SD_id, code_1C are not necessarily globally unique across branches/source servers.

## Variantlar
External ID as primary key; source composite as PK; internal ID with mappings.

## Taklif etilgan yechim
Internal immutable IDs plus scoped tenant/provider/connection/server/branch/entity/ID mappings; quarantine collisions.

## Sabab
Safe migration and avoiding accidental cross-tenant merges.

## Xavflar
Complex mapping, manual conflict review.

## Boshqa modullarga ta'siri
Database, Multi-Tenant, CRM, Catalog.

## Qaror sharti
Owning domain review required; no implementation authorization. See [main architecture](../modules/INTEGRATION_ARCHITECTURE.md).
