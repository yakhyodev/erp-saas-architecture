# ADR-DB-006 — Tenant-aware indexes and measured partitioning

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** mixed tenant OLTP hot paths and long event histories.
- **Variantlar:** blanket indexes/partition all; targeted composite indexes + evidence-based time partitioning.
- **Tanlov:** targeted indexes, keyset pagination, measured partitioning.
- **Sabab:** less write amplification and schema complexity before justified.
- **Xavf:** growing table skew; partition uniqueness traps.
- **Ta’sir:** Reporting, Billing, Inventory, Ops.

## Round 2 concurrency and partition gate (PROPOSED)

Hot tenant/warehouse/SKU/dimension StockBalance uses deterministic row locking or conditional atomic update with short lock duration, contention/backoff metrics and uniqueness per physical movement leg. Two concurrent reservations/dispatch retries cannot oversell under the *as-yet-unapproved negative stock policy*. Posted StockMovement/FinanceJournal/BillingLedger records immutable; reversals are distinct linked append-only facts. Monthly partition indexes alone cannot ensure all-time idempotency: partitioned unique/PK needs partition key, so preserve global replay key in a durable unpartitioned dedupe registry (or explicitly proven equivalent) across archive/detach boundaries. Benchmark tenant-scoped lookup, partial outbox polling indexes, query plans, p95/p99, write amplification, vacuum lag, skew, and keyset pagination before partition adoption. No numeric performance SLO or partition threshold approved. Database §18.4 defines test scenarios.
