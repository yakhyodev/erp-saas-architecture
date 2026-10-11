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
