# LOCALIZATION_ARCHITECTURE.md

- **Version:** 1.0 (2026-10-11)
- **Owner:** Localization Architecture
- **Status:** APPROVED business requirements + PROPOSED technical architecture
- **ADRs:** [001](../adr/ADR-LOC-001.md) · [002](../adr/ADR-LOC-002.md) · [003](../adr/ADR-LOC-003.md) · [004](../adr/ADR-LOC-004.md)

## 1. Scope and approved requirements
Localization is a cross-cutting ERP SaaS capability for UI translation, locale preferences, formatting, Unicode, search and future alphabet migration. Authentication, tenant authorization, database engine selection, legally binding document content and billing calculations belong to their respective modules.

**APPROVED:** Uzbek, English and Russian public interface languages; prepare a hidden future Uzbek alphabet variant; company and branches managed within one SaaS. The proposed technical stack (Go, PostgreSQL, Next.js and Flutter), modular monolith and tenancy implementation are not final approvals.

**PROPOSED:** explicit script-aware locale identifiers, versioned translation catalogs, distinct original business values and derived search aliases, safe fallbacks, tenant-isolated multilingual search, opt-in feature-flagged new-alphabet rollout.

## 2. Locale registry and preference
| Code | Meaning | Exposure | Decision state |
|---|---|---|---|
| `uz-Latn-UZ` | Current Uzbek Latin UI | Public | PROPOSED code |
| `en` | English UI | Public | PROPOSED code |
| `ru` | Russian UI | Public | PROPOSED code |
| `uz-Latn-UZ-x-new` | Future Uzbek alphabet | Internal, OFF | PROPOSED temporary private-use code |
| `uz-Cyrl-UZ` | Cyrillic Uzbek script/search support | Not a separate public UI | PROPOSED scope |

The new alphabet's exact official standard, legal effective date and compulsory migration rules have **not** been verified; no date is assumed.

**PROPOSED preference resolution:** explicit user choice > tenant default > platform default (`uz-Latn-UZ`). Browser/mobile locale may initially suggest, never silently override, a user choice. Language, timezone (IANA), currency (ISO 4217) and regional number display are independent preferences. Switching language cannot modify accounting values, balances or business documents.

**PROPOSED fallback:** exact catalog > compatible same-script base catalog (if present) > `uz-Latn-UZ` > safe visible missing-key handling with telemetry. New alphabet can fall back to current Uzbek Latin; critical missing translations block release. Enterprise restrictions on user locale remain OPEN.

## 3. Translation system
- Stable semantic module keys, e.g. `sales.order.create`, `inventory.stock.available`, `billing.wallet.insufficient`; no hardcoded translated UI strings as identifiers.
- Versioned catalogs and module boundaries, consistent ERP glossary, key deprecation and rename policy, checks for missing/unused keys and placeholder parity.
- Named typed parameters, plural rules and grammatical forms based on reviewed ICU/CLDR/MessageFormat semantics. Exact library selection and version are OPEN.
- Backend/API emits locale-agnostic stable `error_code` and safe params; the presentation surface performs translation.
- Treat interpolation as untrusted; escape user values, prohibit executable markup, allow narrowly controlled rich text only.
- Emails, push, printable documents and PDFs have a template ID, locale and version; snapshots of issued legal/financial documents remain unchanged after catalog updates.
- Lazy loading per module on web; cached versioned baseline catalogs for offline mobile. Cache identity: locale + module + catalog version.
- QA: pseudo-localization, long Russian translations, font and glyph coverage, accessibility labels, missing-key gate, offline fallback and plural validation.

## 4. Original business information invariant
| Business information | Stored/returned policy |
|---|---|
| Customer, product, company and supplier names | Preserve user-entered originals; optionally store an explicitly reviewed localized label separately |
| SKU, barcode, INN, invoice/document numbers | Exact identity; never transliterate or locale-transform |
| Status and error codes | Stable enum/code plus translated presentation |
| Issued document lines and text | Immutable document snapshot; separate allowed localized rendering only |
| Audit | Stable action identifiers and original values; translate only display |

Store raw input where byte-perfect audit requires it, with optional NFC canonical display distinct from raw input. NFKC and transliteration are derived search-only transformations, never destructive updates of original domain fields. Search data must remain tenant scoped.

## 5. Uzbek Unicode and search normalization
**PROPOSED versioned search-only pipeline:**
1. Validate UTF-8 and define strict sanitization/error reporting.
2. Normalize search text to NFC, safe casefolding and whitespace canonicalization.
3. Contextually map apostrophe-like variants in Uzbek letters (`O‘zbekiston`, `O'zbekiston`, `Oʻzbekiston`) without universally deleting punctuation.
4. Generate script-aware Uzbek Latin/Cyrillic search aliases from reviewed mappings; transliteration is not translation and may be ambiguous.
5. Build tokens and rank exact identifier > exact stored name > normalized exact > Latin/Cyrillic-equivalent > prefix > optional fuzzy.
6. Always apply tenant/role/branch authorization to search results, autocomplete, counts, indexes and cache keys.
7. Attach `normalizer_version` to derived search values; support staged reindex, collision metrics and rollback.

Do not merge `ҳ/х`, `қ/к`, `ў/у`, `ғ/г`, `е/ё` indiscriminately. Exact identifier `O'123` must never become another SKU through apostrophe folding. Search should test `Ўзбекистон` / `Узбекистон` and Latin variants with a reviewed ambiguity policy. Original business data must be identical before and after searching. Database-native indexing is the initial candidate for benchmarks; any dedicated search engine selection remains OPEN.

## 6. Formatting
- Store timezone-aware canonical timestamps; localize displays through explicit IANA timezone and chosen locale. Retain canonical audit time.
- Currency amount and code belong to the domain record. Use exact decimal/minor unit arithmetic as separately agreed by Finance; localization is formatting only. **APPROVED:** 1 credit = 10 so'm.
- Use reviewed CLDR formatting for dates, numbers, pluralization and units. Financial import must use an explicit separator convention, never guess comma/dot interpretations.
- Canonical unit codes and machine-readable API values are locale independent. Document export encoding, separators and ISO variants must be documented.

## 7. Hidden future Uzbek alphabet
**APPROVED:** readiness for the new hidden alphabet. **PROPOSED:** private internal registry variant and feature flag `locale.uz_new.enabled` default OFF, excluded from public language choices.

Rollout: dormant infrastructure -> internal QA -> authorized tenant pilot -> separately approved general release. Test catalogs, glyphs, font fallback, transliteration collisions and offline compatibility. Turning flag OFF reverts presentation to existing Latin without modifying records. Never run an irreversible automatic rewrite of names, invoices, audit values or partner identifiers. Feature-flag changes need audit history. External standardized tag selection remains OPEN.

## 8. Security, offline and performance
A locale never changes permissions. Catalog arguments must be escaped. Search isolation applies to suggestions and metrics as well as results. Offline localization caches a minimum valid catalog and uses versioned update policy; sync must preserve domain originals. Benchmark catalog size, switching latency, query ranking, collision rates and index overhead before committing to SLOs or expensive search services. Layout QA covers text expansion and accessibility.

## 9. Cross-module contracts
| Related module | Contract |
|---|---|
| Master Architecture | Shared localization capability and original business invariants |
| UI/UX / Frontend | User language selector, catalogs, fallback, accessibility |
| Backend / API | Stable error code + params, locale-independent contract |
| Multi-Tenant / Security | Tenant isolation, branch visibility, audit |
| Database | Original and derived fields; normalizer version; collation/index benchmarks |
| CRM / Sales / Inventory | Display names, language labels, exact SKU, printed document snapshots |
| Finance / Billing | Exact money values, approved document templates and issue-time snapshots |
| Mobile / Offline | Cached catalogs, offline search compatibility |
| Integrations | Stable canonical identifiers and explicit external locale policy |

## 10. ADR registry (not yet approved)
- [ADR-LOC-001 — Locale identity and fallback](../adr/ADR-LOC-001.md): **PROPOSED**
- [ADR-LOC-002 — UI catalogs and business originals](../adr/ADR-LOC-002.md): **PROPOSED**
- [ADR-LOC-003 — Versioned Uzbek search](../adr/ADR-LOC-003.md): **PROPOSED**
- [ADR-LOC-004 — Hidden alphabet rollout](../adr/ADR-LOC-004.md): **PROPOSED**

## 11. OPEN QUESTIONS
1. Legally valid new Uzbek alphabet, exact orthographic mapping, effective date and mandatory migration scope? No official date is asserted here.
2. Fixed or configurable platform default locale?
3. Full `uz-Cyrl-UZ` UI required, or script-aware search only?
4. May enterprise tenant restrict user interface language?
5. What are statutory language/template rules for invoices and fiscal documents?
6. Ownership of reviewed transliteration mappings, collisions and imported encodings?
7. Real query scale, acceptable false positives and measured latency targets?
8. Final ICU/CLDR/MessageFormat library versions after stack decisions?

## 12. Shared-document synchronization proposals — NOT APPLIED
- `DECISIONS.md`: register ADR-LOC-001..004 as PROPOSED only.
- `MASTER_ARCHITECTURE.md`: reference cross-cutting localization and separate original-vs-derived data.
- `REQUIREMENTS.md`: record APPROVED three languages and hidden-alphabet readiness; proposed cross-script search, formatting and safe API conventions separately.
- `MODULE_REGISTRY.md`: link Localization to UI/UX, Backend, Database, Multi-Tenant, Security, Finance, CRM, Sales, Inventory, Mobile and Integrations.
- `OPEN_QUESTIONS.md`: link eight unresolved questions, including new alphabet effective date.
- `CHANGELOG.md`: 2026-10-11 architecture v1.0 proposal (NOT an approval).

## 13. Codex specification — DESIGN ONLY, NO CODE
Prepare an implementation design and contract specification covering: locale resolution/fallback; semantic catalogs and key validation; typed placeholders and plural rules; stable localized error-display mapping; versioned email/PDF templates; original-vs-optional-localized-vs-derived search data; conservative versioned Uzbek apostrophe and Latin/Cyrillic normalization; tenant-scoped authorized ranking and reindex; default-OFF alphabet feature flag with reversible pilot/rollback; currency/date/time/number formatting; offline catalogs; thorough test matrix.

**Acceptance criteria:** Switching locale never alters balances or issued documents; external API IDs and error codes remain stable; exact SKU ranks above fuzzy matches; search has no tenant leak; critical translation omissions fail checks; original text is preserved; offline language behavior remains usable; hidden alphabet is invisible while OFF and rollout is reversible.

**Restrictions:** Do not generate application code or migration SQL yet. Do not invent legal deadlines, transform originals destructively, declare proposals APPROVED, or silently finalize the stack. Identify conflicts with other modules.

## 14. References
- BCP 47 (RFC 5646): https://www.rfc-editor.org/rfc/rfc5646.html
- Unicode UAX #15: https://www.unicode.org/reports/tr15/
- Unicode CLDR: https://cldr.unicode.org/

## 15. Cross-module reconciliation: search, UX and alphabet quality gate (PROPOSED, 2026-10-12)

This section reconciles the failed concurrent `main` update without changing the approved languages. Canonical business identifiers, document numbers, tenant IDs, product SKU, accounting values and original business names **never change** when the display locale changes.

| Contract | Source of truth | PROPOSED acceptance test |
|---|---|---|
| UI locale selection | Explicit user setting, then tenant and platform defaults (precedence proposed) | Web/mobile/offline same preference; unsupported and missing keys render deterministic safe fallback, not silent data corruption |
| API errors and events | Backend stable `error_code`, `schema_version`, typed safe params | Changing locale does not alter authorization, event identity or idempotency |
| Original vs search-normalized text | Business owner retains original, Search stores versioned derived aliases | Uzbek Latin apostrophe/glyph variants and Cyrillic queries are searchable; exact SKU/barcode dominates fuzzy alias; no false canonical ID collisions |
| Security in global search | Security permission enforcement + verified tenant/company/branch scope | Search counts, suggestions, filters, caches and background reindex jobs never leak unauthorized tenant/branch items |
| Indexed version migration | Search index `normalizer_version` + offline cache version | Shadow index/backfill/compare/atomic cutover/rollback; old/new version compatibility tests and no loss of original text |
| Hidden new Uzbek alphabet | Localization feature registry, server-side feature access and versioned bundle | OFF by default, absent from public menu; internal pilot opt-in only; old content remains accessible, fallback tested |
| Accessibility and usability | UX acceptance + localized catalogs | Screen reader labels, keyboard focus, mobile touch, long Russian strings, plural/currency, date/number formatting, pseudo-localization and offline missing-key tests |

**Responsibilities:** Localization owns locale catalog, script mappings and normalization proposal; UX owns task-first search experience; Security owns permission semantics; Backend/API owns stable contracts; Database/Search owns index consistency and tenant-scoped query execution; Mobile owns offline cache state. Transliteration is never an unreviewed mutation of legal, ledger or stock facts.

**Release gates (OPEN):** platform default locale choice, approved normalization mapping and alias collision thresholds, full Cyrillic Uzbek UI versus search-only support, enterprise language restrictions, search latency/usability numeric targets, and final publication rules for future Uzbek alphabet. All proposed test thresholds and locale identifiers still need corresponding ADR / explicit approval as applicable.

## 15. Round 2 correction — AR-012, MR-17, MR-20 (2026-10-12)
**Status: PROPOSED contracts; QA NOT EXECUTED.** Round 2 [audit](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md) identifies MR-17 authorization/freshness and MR-20 accessibility/device/locale readiness as OPEN. UX owns workflow completion and UI quality; Security/Backend own authoritative authorization; Localization owns linguistic transformations and catalogs. **APPROVED**: uz/en/ru interface and hidden future Uzbek-alphabet readiness only. No new alphabet public exposure or ADR approval is implied.

### 15.1 Registry, effective preference and catalog protocol (PROPOSED)
- Public registry entries: `uz-Latn-UZ`, `en`, `ru` (code choices PROPOSED). `uz-Cyrl-UZ` is search-script support, not a newly approved fourth UI. `uz-Latn-UZ-x-new` remains an **internal provisional alias**, excluded from public registry, API language discovery, SEO/links and user-visible preference selectors.
- Effective language is resolved at a **trusted session/scope**: explicit active-user preference (only if enabled and authorized) → tenant-preferred public locale → deployment/platform default (exact code OPEN). Unsupported, stale or disabled preference resolves to permitted fallback; no feature flag grants data permission. Browser/device preference is initial suggestion only. Tenant/branch switches re-evaluate context; optional branch-specific language override is OPEN, not assumed.
- Expose a **presentation-only** resolution contract: `requested_locale`, `effective_locale`, `fallback_reason`, `catalog_release` and `scope_revision` (field names PROPOSED); never trust supplied locale or tenant as authentication context. The backend retains tenant-independent stable error/status codes; clients localize labels.
- Catalog identity: `locale + module + catalog_release + contract_major` (and build integrity digest); translation key identity `module.semantic.action`, with immutable semantic meaning, typed named parameters and tested plural/select branches. A key rename requires an explicit compatibility alias/deprecation window, not silent repurposing. Clients reject incompatible major versions and use a signed/verified last-known-good compatible bundle; missing critical safety/finance text blocks release rather than showing misleading fallback. Keep a deterministic per-key fallback chain: effective locale → compatible approved fallback → non-sensitive safe generic error with diagnostic key; no mixed version of critical workflow messages.
- All public locale choices receive coverage for loading, empty, error, confirmation, keyboard navigation and screen-reader strings. Preserve a shared glossary between web and mobile. Backend error code + typed params is stable across devices; raw error payload must not expose private records during fallback.

### 15.2 Immutable domain invariants and reversible search (PROPOSED)
Locale selection, normalization, transliteration and search reindex **MUST NOT mutate** Product ID, SKU, barcode, document number, INN, financial amount/currency/credit, tenant/company/branch ID, actor, domain event ID, business occurrence ID, stable operation ID, audit history or issued document snapshot. Identity matching is exact/canonical per owning domain, **never** based on localized display strings.
- Store raw originals when evidentiary preservation is needed. Derive separate search-only NFC/casefold/whitespace forms and contextual Uzbek apostrophe aliases; preserve `oʻ/gʻ` distinct from plain `o/g`, and do not apply indiscriminate NFKC or punctuation stripping to IDs.
- Generate Latin/Cyrillic Uzbek transliteration aliases with explicit `normalizer_version` + `mapping_version` and provenance (`original`, `normalized`, `script_alias`, `reviewed_synonym`). English/Russian use language-aware tokenization and safe Unicode casefold, **not** Uzbek transliteration applied to all languages. Use language detection only as a hint, not destructive transformation.
- **Collision strategy variants (OPEN user/architecture choice):** (A) conservative no-alias on ambiguous pairs; (B) multiple ranked candidates with explicit original-label and scope; (C) manually curated exception dictionary. Interim PROPOSED fail-safe: never silently merge identifiers/records, rank exact original/exact ID first, surface ambiguities as separate authorized candidates, count false positives and hold highly ambiguous aliases for review. Examples include `ҳ/х`, `қ/к`, `ў/у`, `ғ/г`, `е/ё` and apostrophe collisions.
- Index/reindex: tenant-scoped immutable source ID references, authorization context, `index_schema_version`, `normalizer_version`, `mapping_version` and source revision. Build a shadow generation, replay changes after snapshot, compare counts/collision/authorization tests, switch pointer atomically per tenant partition, retain previous generation for rollback; never overwrite originals. Freshness and acceptable lag thresholds remain OPEN and require measurement.

### 15.3 Authorized search coverage matrix (MR-17, PROPOSED)
| Surface | Mandatory trusted scope / enforcement | Cache / migration / negative test |
|---|---|---|
| Results and detail links | Backend verifies current tenant, company, branch, membership, resource permission before disclosure | Unauthorized IDs excluded; reauthorize detail open |
| Total counts and pagination | Count **only authorized** result set under same filter/revision | Forbidden record does not change count |
| Autocomplete, spelling and suggestions | Authorized projection, no tenant-global raw dictionary exposing sensitive terms | Test tenant A text absent in B suggestions |
| Facets, aggregations and filters | Authorization before aggregation, no inference through bucket totals | Test zero/hidden bucket leakage |
| Recent query/history/saved searches | User+tenant+scope ownership, bounded retention and explicit cleanup | Switch/revoke hides history; no cached B data in A |
| Response cache / CDN / local mobile | Partition by tenant, authorized principal/scope and grant-version; sensitive responses private/no-store where needed | Grant revocation / tenant switch invalidates or fails closed |
| Search indexes / reindex / background workers | Scoped source readers and fenced generation, read-time server authorization remains authoritative | Stale index and moved/revoked membership fail closed |
| Locale switch / alias expansion | Locale changes display/aliases only; never broadens scope | Same authorized canonical ID set unless query semantics expressly alter matching |

**No client-side hiding is permission enforcement.** Backend and Security must choose authoritative grant-version/revocation protocol (AR-013 OPEN). If authorization context cannot be refreshed, deny sensitive requests, including offline cached cross-tenant search. Search index placement and engine remain Database/Infrastructure OPEN.

### 15.4 Web, mobile and offline consistency (PROPOSED)
- Web and mobile consume catalog bundles compiled from the **same semantic key registry and contract major**; release digest, resolved locale and fallback reason are observable without logging private translation arguments.
- Offline app pins last-known-good verified bundle per user/tenant; app can display locally assigned cached content only within its last valid scope, with offline age and revocation limitations indicated. On reconnect, revalidate membership, branch, device, current grant-version and bundle compatibility before fresh server queries/sync.
- Locale switch while offline may use installed compatible bundles only; otherwise use safe current-language fallback and communicate that language download is needed. Never silently show a half-loaded mixture of finance/permission critical text. Locale cache purge on sign-out, tenant switch and revoked access follows Mobile/Security policy (remote immediate wipe cannot be guaranteed).
- Errors remain stable codes/typed non-sensitive arguments independent of device language; unsupported catalog major uses last known compatible bundle and explicit fallback, not guessed translations.

### 15.5 Hidden alphabet rollout and rollback (PROPOSED)
Default `locale.uz_new.enabled = OFF`. Public registry, preference UI, public API metadata, notification templates, exports and SEO must never expose the option while OFF. Internal preview requires explicit role-authorized tester + separate environment/tenant allowlist, audit event, compatible private catalog and internal-only route; pilot and general release require future explicit approval **not granted here**.
**Gates:** legal/orthography verified (date OPEN); glossary & typography signed off; translation parity, cross-device/offline QA, security leaks, search collision regression, issued-doc immutability, index rollback and support recovery completed. **Rollback:** disable flag, invalidate bundles and alias generation, revert active search index pointer/catalog release, maintain originals, preserve audit and notify opted-in testers; keep rollback functional before any pilot.

### 15.6 Cross-device and accessibility QA matrix (design only; NOT RUN)
| Scenario | Uzbek `uz` | English `en` | Russian `ru` | Hidden variant |
|---|---|---|---|---|
| Desktop web keyboard/tab/focus/search | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Mobile small/large screen, portrait/landscape, touch targets | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Offline first launch with verified cached bundle / fallback | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Dynamic text scaling, wrapping, long labels, Cyrillic glyph/font | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Screen reader labels, live alerts, errors and dialog focus | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Search Latin/Cyrillic/apostrophe, exact ID and collision ranking | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Tenant/branch switch, revocation, autocomplete/count/facet denial | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Catalog skew, missing key, stale bundle and offline-to-online update | REQUIRED | REQUIRED | REQUIRED | Internal QA only |
| Document number/money/ID invariance after locale switch | REQUIRED | REQUIRED | REQUIRED | Internal QA only |

Desktop browsers, mobile OS/device list, assistive technology combinations, accessibility conformance version/level, sample datasets, query latency and task completion benchmarks are **OPEN** with UX/Mobile/Security. Do not claim any pass. Use seeded tenant A/B, separate branches, revoked user/device, ambiguous apostrophe pairs, long Russian labels, untranslated key and malformed UTF-8 fixtures; tests must check both outcome and no disclosure.

### 15.7 Release quality gate and ownership (MR-17/MR-20)
**PROPOSED block release** on critical untranslated/error messages, mismatched typed placeholders, unsupported offline bundle contract, public hidden-alphabet exposure, unauthorized result/count/suggestion/facet/history, index cross-tenant leak, stale-grant bypass or transformed immutable identifier/amount. UX signs off task flows and keyboard/screen reader acceptance, Localization catalog and linguistics, Security/Backend authorization, Mobile offline integrity and Database reindex integrity; none is APPROVED or tested yet.

**OPEN decision packet:** platform fallback/default locale; ambiguous alias collision policy; standalone Uzbek Cyrillic UI need; legal alphabet standard/start date; catalog runtime/ICU version; search freshness target and index owner; tenant enterprise locale override; quantitative UX/a11y/browser/device acceptance baselines. Central registry synchronization is for Master owner, **not applied in this chat**.
