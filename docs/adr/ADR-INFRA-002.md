# ADR-INFRA-002: PostgreSQL resilience evolyutsiyasi

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Avoid data loss and unsustainable HA cost.

## Ko‘rib chiqilgan variantlar
Managed PostgreSQL HA; self-managed primary/standby; single primary with recovery.

## Tanlangan yechim — PROPOSED
First verify encrypted off-site backups, WAL/PITR and restore; add managed HA or tested standby when downtime constraints demand.

## Sabab
Replication is not a backup; incremental resilience.

## Xavflar va cheklovlar
Primary downtime, replica lag and split-brain risk. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Database, Finance, Billing, Security.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.
