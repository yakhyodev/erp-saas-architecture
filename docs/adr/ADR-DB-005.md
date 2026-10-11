# ADR-DB-005 — Append-only finance/billing ledgers

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** auditable account and credit balances with corrections.
- **Variantlar:** overwrite-only balance; mutable history; immutable ledger + summary.
- **Tanlov:** immutable ledger and controlled reversal + reconciled summaries.
- **Sabab:** traceability, deterministic balance rebuild, concurrent correctness.
- **Xavf:** growth, backfill/partition difficulties, correction workflows.
- **Ta’sir:** Finance, Billing, Reporting, Audit.
