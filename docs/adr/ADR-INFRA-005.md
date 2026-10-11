# ADR-INFRA-005: Tenant placement abstraction

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Future enterprise isolation without paying for dedicated databases upfront.

## Ko‘rib chiqilgan variantlar
Fixed shared DB; dedicated DB for all; registry-based placement.

## Tanlangan yechim — PROPOSED
Shared-first if other architects agree; registry-based tenant-to-cluster/database placement and migration path.

## Sabab
Enables dedicated DB and future sharding without upfront complexity.

## Xavflar va cheklovlar
Migration consistency, registry availability and global-query cost. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Multi-Tenant, Backend, Database, Billing, Security.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.
