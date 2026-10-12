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

## Round 2 AR-003 / MR-14 backup and DR acceptance — 2026-10-12 (PROPOSED)

Compare snapshot-only (cheap but incomplete PITR), base backup + WAL off-site and restore drills (recommended candidate), and cross-region standby plus independent backups (more operational/network expense; replication is not backup). Verify complete WAL chain and manifest/checksums, encryption and key recovery, deliberate missing-segment alarm, restore to at least distinct controlled timestamps, ledger/stock/object consistency, and per-tenant non-disclosure. For shared-database tenant recovery use isolated restore plus scoped extract and verification, **not wholesale PITR of other tenants**. B/C additionally require exclusive writer fencing, split-brain, failover/failback, replica lag and business event idempotency drills. Store observed data loss/recovery timing as **test evidence**, not achieved target claims. Retention, RPO/RTO, DR/backup region, legal hold and budget all **OPEN** pending user and Security/Database approval. See [Infra §19](../modules/INFRA_ARCHITECTURE.md).
