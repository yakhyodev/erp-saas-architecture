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

## Round 2 live Billing §21 compatibility — 2026-10-12 (PROPOSED)

Billing has recorded company-scoped BillingAccount/Wallet, anniversary-based calendar month, activated daily-prorated Seat, server-effective month with current-period corrections and marginal progressive tiers as **SELECTED_PENDING_APPROVAL**, not APPROVED. Sales/Inventory must not implement them as rules. It only supplies immutable source_company_id/branch_id, document type/transition/line and quantity metric, canonical business_occurrence_id distinct from transport event_id, source revision, domain_effective_at, server_accepted_at and optional untrusted device_occurred_at, linked original occurrence for return/reversal, and external-document mapping provenance.

Billable type/transition; base fee per canonical order versus split dispatch; cancellation/return/refund; imported historical backfill; amendment delta; tariff rounding; billing period/late event and payer liability remain **OPEN**. Billing alone resolves effective BillingAccount mapping, determines eligibility and rates genuine charges. Duplicate transport re-envelopes must not create duplicate economic charge effects; reversal never automatically refunds. The Sales/Inventory §13 matrix and Billing §20.4/§21 are non-approved coordination documents.

**Approval dependencies:** Billing 01, Sales 05, Integrations 12, Multi-Tenant 02, Mobile 09 and user confirmation; preserve AR-001/009/010 OPEN.
