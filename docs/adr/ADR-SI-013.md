# ADR-SI-013: Chargeable document occurrence boundary

- **Status:** PROPOSED — not an approval
- **Date:** 2026-10-11
- **Harmonization:** [SALES_INVENTORY.md §11](../modules/SALES_INVENTORY.md)

## Muammo
AR-010: per-document Business Usage APPROVED but charged document type/transition and refund/reversal NOT defined.

## Ko‘rib chiqilgan variantlar
A charge draft; B charge confirmation; C charge posting; D charge finalization; per-type configurable trigger.

## Tavsiya etilgan yechim
Document owner emits immutable non-monetary lifecycle candidates with stable canonical document and occurrence IDs; Billing exclusively decides eligible tariff/version, deduplicates, rates real expense and adjusts. Trigger per type remains OPEN. Split delivery, import, partial and reversal must be explicitly defined; no automatic refund inferred.

## Sababi
Avoid double billing, decouple ERP financial postings from SaaS credits and preserve audit.

## Xavflar va mitigatsiya
Wrong chargeable transition, imported-history duplicates, retroactive offline month (AR-009), wallet/payer scope (AR-001) all OPEN.

## Boshqa modullarga ta’siri
Billing, Sales, Purchasing, Integrations, Mobile, Multi-Tenant, Finance.

## Open / approval gate
01+05+12/user approve type×transition matrix, partial/refund and imported history. AR-010 remains OPEN.

**Existing APPROVED business invariants untouched. Go/PostgreSQL/Next.js/Flutter/modular monolith remain PROPOSED. No code.**
