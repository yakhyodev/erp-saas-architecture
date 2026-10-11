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
