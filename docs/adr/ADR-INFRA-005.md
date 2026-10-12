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

## Round 2 MR-15 tenant placement and recovery gate — 2026-10-12 (PROPOSED)

Use a tenant_id → placement_id + placement_epoch abstraction only if Multi-Tenant/Database/Security approve. Before dedicated DB or tenant sharding, test hot-tenant impact, DB pool fairness and routing under failover, migration cutover and single-writer fencing; verify idempotent Billing/stock consequences. Compare pooled, dedicated and sharded operating cost including duplicated backup, retention, region, restore and monitoring overhead. A per-tenant restore from a pooled DB requires isolated point-in-time recovery and scoped data/reference reconstruction; avoid cross-tenant rollback. Sharding/dedicated deployment is not MVP mandatory; triggers and region are **OPEN**. See [Infra §19](../modules/INFRA_ARCHITECTURE.md).
