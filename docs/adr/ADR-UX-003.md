# ADR-UX-003 — Universal search + commands

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** ma'lumot turiga qarab turli sahifalarda izlash kerak.
- **Variantlar:** har modulda alohida qidiruv; yagona search natijalari; search + command palette.
- **Tanlov:** ruxsatga asoslangan global search + alohida belgilangan commands.
- **Sabab:** ma'lumot yoki amalni kirish nuqtasidan mustaqil topish.
- **Xavf:** indeks kechikishi, permission leak, qidiruv xarajati.
- **Ta'sir:** Backend, Database, Security, audit, Resource Usage (pricing doirasini billing moduli belgilaydi).

## MR-17 search acceptance — PROPOSED

Server-authorize product, customer, document, order, payment, employee and command groups, including counts, suggestions, facets, snippets, recents, saved queries, exports and details. Recheck on click/action; cache and index scope/version isolated by tenant/membership/branch. Revoked/deleted index entries cannot leak even while rebuilding; reindex must be tenant-scoped, auditable, resumable, with freshness reporting. Test cross-tenant prefixes/IDs, cache races, role revoke, indexing lag, languages and diacritics. Index lag, search cost and operational ownership remain OPEN. See [UX §16](../modules/UX_ARCHITECTURE.md), [Security](../modules/SECURITY_ARCHITECTURE.md), [Localization](../modules/LOCALIZATION_ARCHITECTURE.md).

## Round 2 UX correction — AR-012 / MR-16/17/20 (PROPOSED)

Round 2 MR-17: Authorized search covers result titles, counts, facets, snippets, autocomplete, recents, history, saved queries, related records, commands, export and exact-ID lookup. Backend intersects tenant/company/branch/record grants and rechecks on open; scoped caches/index versions and revocation suppression; reindex freshness contract remains OPEN. Test cross-tenant same prefix, stale cache, deletion and locale normalization.

Details: [UX architecture §17](../modules/UX_ARCHITECTURE.md). No tests performed; no approved new technical decision.
