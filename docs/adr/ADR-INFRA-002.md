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

## Round 2 AR-003 / MR-14 refinement — 2026-10-12 (PROPOSED)

**Staged model aligned with Database ADR-DB-007:** A = single-primary, encrypted off-site base backup + continuous WAL/PITR + verified isolated restore (**NOT HA**); B = primary+standby with monitored replication, single writer, fencing, controlled failover and failback; C = measured automatic failover only after B tests/operations justify it. Managed HA, self-managed standby and recovery-first remain alternatives with cost/ops/latency differences. **OPEN user decision:** launch A/B, RPO/RTO, uptime SLO/SLA, region/residency, sizing and budget; none approved. Stage A/B gate: test backup chain, WAL gap alarm, replay and restore integrity; for B test promotion under old-primary fencing, split-brain network partition, connection recovery and finance/billing/stock invariants. Stage C gate: repeated measured drills, on-call capacity and approved availability/cost case. Evidence not executed; no achieved recovery or latency claims. See [Infra §19](../modules/INFRA_ARCHITECTURE.md) and [Database §18.5](../modules/DATABASE_ARCHITECTURE.md).
