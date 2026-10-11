# ADR-INFRA-004: Backup va DR boshlang‘ich productiondan

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Transactional ERP cannot rely on snapshots that have not been restored.

## Ko‘rib chiqilgan variantlar
Periodic snapshot only; PITR plus off-site backup and restore drills.

## Tanlangan yechim — PROPOSED
Require off-site encrypted backups, WAL/PITR and tested isolated restore before production.

## Sabab
Restorability matters more than the appearance of backup success.

## Xavflar va cheklovlar
WAL gaps, credential loss, retention cost. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Database, Finance, Billing, Security.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.
