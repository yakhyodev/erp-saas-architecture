# ADR-DB-007 — WAL/PITR and HA with practiced restores

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** loss recovery, infrastructure failure, accidental data mutation.
- **Variantlar:** periodic dumps only; base backup+WAL/PITR; per-tenant physical databases.
- **Tanlov:** encrypted base backup + WAL archive/PITR + standby + isolated restore practice.
- **Sabab:** point-in-time recoverability and availability.
- **Xavf:** WAL archival failure, restore time, shared tenant extraction complexity.
- **Ta’sir:** Infrastructure, Security, Compliance, Support.
