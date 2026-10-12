# ADR-LOC-001 — Locale identity and fallback
- Date: 2026-10-11
- Status: **PROPOSED**
- Owner: Localization Architecture

## Problem
Three approved interface languages and a hidden future Uzbek alphabet require stable, explicit locale identities and safe fallback without confusing language preferences with currency, timezone or tenant access.

## Considered alternatives
1. Generic `uz` only.
2. BCP 47-compatible language/script/region tags and an internal provisional private-use variant.
3. Backend translating all presentation strings.

## Proposed decision
Use `uz-Latn-UZ`, `en`, `ru` as proposed public identifiers and `uz-Latn-UZ-x-new` internally behind a default-OFF feature flag. `uz-Cyrl-UZ` is initially only a proposed script/search capability, not an approved fourth UI language. Locale preference precedence: explicit user selection > tenant default > platform default (proposed `uz-Latn-UZ`); missing-key fallback must be deterministic and observable. Locale, timezone, currency and numeric format are separate concepts.

## Rationale
Supports script evolution and auditable preferences without forcing irreversible data conversion.

## Risks
Private-use tag cannot be assumed to work across external services; translation fallback may expose missing content. Default locale and enterprise policies remain open.

## Cross-module impact
UX, Backend API, Mobile Offline, Multi-Tenant, user settings, integrations.

## Approval boundary
Only language support and hidden future alphabet requirement are APPROVED. All identifiers, resolution details and fallback behavior above are PROPOSED.

## Round 2 proposed contract refinement — AR-012 / MR-20
Effective locale: explicitly authorized user preference → tenant preference → **OPEN platform default**. Validate registry and flag for each scope switch; browser/device is only first-use hint. Public choices are uz/en/ru; proposed tags do not imply final BCP 47 standard adoption. Hidden internal tag never returned in public locale enumeration while disabled. Locale resolution may expose presentation-only requested/effective/fallback reason/catalog release metadata without becoming identity authority. Translation fallback must respect compatible catalog contract-major and block critical unlocalized messages. Web/mobile/offline must agree on resolved preference and versioned glossary. **OPEN**: exact platform default, branch override, tenant restriction and locale-code canonicalization. Previous PROPOSED status unchanged; no test performed.
