# ADR-LOC-002 — UI translation catalogs and original business data
- Date: 2026-10-11
- Owner: Localization Architecture
- Status: **PROPOSED**

## Problem
The ERP SaaS provides multiple interface languages but automatically translating or rewriting user-entered product/customer names and historical financial/legal documents could corrupt meaning, audit records, external identifiers and search.

## Considered alternatives
1. Translate and overwrite all database strings per selected UI language.
2. Copy each business record into locale-specific versions.
3. Preserve original business data and maintain independently versioned presentation catalogs, optional reviewed localized labels and search-only derived fields.

## Proposed decision
Select option 3. Keep original customer/product/company names and issued document snapshots intact; exact SKU, INN, barcode and document identifiers are never transliterated. Render UI labels from semantic module-scoped keys with stable named parameters and versioned catalogs. Optional manually approved alternate labels may be stored in separate fields. Derived normalization and transliteration must never overwrite originals. Backend returns stable `error_code` and parameters for localization at the UI. Retain raw input where byte-perfect evidence is required.

## Rationale
Protects financial/legal correctness, API contracts, auditability and future reindexing. A locale switch cannot change real monetary values or record identity.

## Risks
Some business data remains visible in its original language; reviewing localized labels and maintaining dictionaries requires governance.

## Cross-module impact
Database, Sales, Inventory, CRM, Finance, Billing, Backend/API, UX, Audit, Integrations.

## Approval boundaries
**APPROVED**: three interface languages, a hidden future Uzbek alphabet and existing ERP business rules. **PROPOSED**: the architectural data separation, catalog format and manually reviewed alternate-label workflow. Final acceptance requires explicit architecture approval.

## Round 2 proposed catalog and identity contract — MR-20
Keys use immutable module-scoped semantic names with catalog release, contract major, integrity digest, typed placeholders, plural/select tests and compatibility/deprecation windows. Web and mobile must share key semantics; offline pins last verified compatible release. Stable backend error codes and sanitized typed parameters are language-independent. UI/localized print templates never mutate original business records, issued snapshots, Product ID, SKU, barcode, document number, INN, tenant/company/branch IDs, financial amount/currency, domain event ID, business occurrence ID or offline operation ID. Translation/sanitization cannot authorize additional records. Catalog skew, critical missing translations, unsafe interpolation, offline fallback and locale-switch immutability are **design release gates, NOT TESTED**. Language registry and runtime/ICU implementation remain PROPOSED/OPEN; existing ADR status unchanged.
