# ADR-LOC-003 — Versioned Uzbek search normalization
- Date: 2026-10-11
- Status: **PROPOSED**
- Owner: Localization Architecture

## Problem
Users may type Uzbek names in Latin, Cyrillic and multiple apostrophe forms. An overly aggressive fold creates collisions, changes identifiers and can leak cross-tenant data.

## Considered alternatives
1. Exact-only queries.
2. Global punctuation/diacritic removal and irreversible transliteration.
3. Preserve originals and build tenant-scoped, versioned search aliases with conservative ranking.

## Proposed decision
Validate UTF-8; generate NFC and context-aware search-only apostrophe/case/whitespace normalization; produce reviewed Latin/Cyrillic aliases with `normalizer_version`. Rank exact identifier > original exact > normalized exact > script equivalent > prefix > optional fuzzy. Do not normalize stored SKU, INN, barcode, document numbers or user originals. Restrict all results, counts, suggestions and caches by tenant and authorization. Support staged reindex, versioned tests and rollback.

## Rationale
Improves discovery while retaining data fidelity and predictable security behavior.

## Risks
Ambiguous transliteration, false positives, growing index size and reindex cost. Collisions such as Uzbek `ҳ/х`, `қ/к`, `ў/у`, `ғ/г` need explicit testing; do not merge blindly.

## Cross-module impact
Database and scaling, CRM, Inventory, Sales, Mobile Offline, Security, Multi-Tenant.

## Approval boundary
Search normalization details and search engine choice are PROPOSED, not APPROVED. Benchmark and mapping review remain required.

## Round 2 proposed collision, index and authorization contract — MR-17
Derive separate NFC/casefold/whitespace, context-aware Uzbek apostrophe and reviewed Latin/Cyrillic aliases while preserving originals. Keep `normalizer_version`, `mapping_version`, `index_schema_version`, source revision and alias provenance; never normalize identifier equality or financial value. Russian/English tokenization does not inherit Uzbek transliteration indiscriminately. **OPEN collision policy variants**: conservative omit ambiguous alias; return distinct ranked authorized candidates; curator-maintained exceptions. Interim proposed default: exact identifiers/original names first, distinct candidates not silent merges, ambiguous aliases held for review.

All surfaces require backend-side authorization over tenant/company/branch/resource: results, counts, autocomplete, facets, query history, saved search, local/CDN cache and index/reindex workers. Re-authorize read at query/detail and fail closed for stale grant/tenant switch; permission cache version contract belongs to Security/Backend (AR-013). Rebuild index in tenant-scoped shadow generation, replay changes, test collisions/no-leaks, switch generation pointer and allow rollback. Test unauthorized counts, suggestion inference, stale index, permission revoke, tenant A/B and cross-branch scope. **No tests run; search engine/freshness SLO OPEN.** ADR remains PROPOSED.
