# ADR-DB-007 — WAL/PITR and HA with practiced restores

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** loss recovery, infrastructure failure, accidental data mutation.
- **Variantlar:** periodic dumps only; base backup+WAL/PITR; per-tenant physical databases.
- **Tanlov:** PROPOSED staged resilience: Stage A encrypted base backup + WAL archive/PITR + verified isolated restore, **without claiming HA**; Stage B monitored standby + failover/fencing only when approved availability/SLO/RPO/RTO require it. Old initial-standby proposal remains an alternative, not a binding decision.
- **Sabab:** point-in-time recoverability and availability.
- **Xavf:** WAL archival failure, restore time, shared tenant extraction complexity.
- **Ta’sir:** Infrastructure, Security, Compliance, Support.

- **Approval gate:** initial production topology and numeric SLO/RPO/RTO remain OPEN to explicit user decision; Database and Infrastructure module docs must match.

## Round 2 HA stage compatibility with Infrastructure — AR-003 (PROPOSED)

**Stage A:** single PostgreSQL primary + encrypted offline/off-site base backups, continuous WAL/PITR, verified isolated restores and restore access rehearsal. It is **not high availability**; primary crash can interrupt service. **Stage B:** primary/standby streaming replication, lag monitoring, single-writer endpoint, fencing/split-brain prevention, rehearsed promotion/failback when approved availability requirements demand. Optional future automatic failover/read routing is not committed. The old initial standby requirement is only a historical PROPOSED alternative. RPO, RTO, uptime SLO, retention, deployment budget, synchronous vs asynchronous replication, managed hosting and initial Stage A/B selection are OPEN for user approval. No failover/restore verification is claimed completed. Cross-reference Infrastructure §18 and Database §18.5.
