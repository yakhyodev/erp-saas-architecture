# ADR-LOC-004 — Hidden future Uzbek alphabet rollout
- Date: 2026-10-11
- Status: **PROPOSED** implementation of **APPROVED** readiness requirement
- Owner: Localization Architecture

## Problem
The ERP SaaS must prepare a hidden future Uzbek alphabet variant although exact orthography, legal force, effective date and mandatory migration rules are not established in this architecture project.

## Considered alternatives
1. Force a new alphabet and mass-rewrite records immediately.
2. Defer all readiness until regulation.
3. Prepare hidden catalogs/registry with controlled, reversible activation.

## Proposed decision
Maintain a dormant internal locale variant with feature flag `locale.uz_new.enabled` default OFF and no public language-selection entry. Stages: dormant -> internal QA -> tenant pilot -> explicit reviewed general release. Preserve current Uzbek Latin fallback and original business records. No unreviewed mass transliteration. Record feature-flag changes in audit history and preserve stable integration values. Treat `uz-Latn-UZ-x-new` as provisional internal tag only.

## Rationale
Meets preparedness requirement while preventing unapproved legal assumptions and nonreversible data changes.

## Risks
Catalog duplication, font/test overhead, uncertain transliteration mapping, external tag incompatibility.

## Cross-module impact
UX, Mobile, Backend, Security, Product governance, Integrations, Finance legal templates.

## Open legal question
The officially enacted orthography, effective date and compulsory scope must be verified separately and recorded in OPEN_QUESTIONS; no date is asserted here.

## Approval boundary
Hidden readiness is APPROVED; flag semantics, rollout phases and locale code remain PROPOSED.
