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
