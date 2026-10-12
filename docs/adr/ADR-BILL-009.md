# ADR-BILL-009 — Chargeable document lifecycle boundary

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Billing with Sales & Inventory and Integrations
- **Issue:** AR-010, MR-04
- **Reference:** [BILLING_ARCHITECTURE §19.2](../modules/BILLING_ARCHITECTURE.md#192-ar-010--mr-04--document-type--transition--charge-policy)

## Muammo
Business Usage hujjat asosiy narxi va hajm oshganda qo'shimcha kredit **APPROVED**, ammo qaysi hujjat turi va status o'tishi charge yaratishi OPEN. Sales, stock posting, ERP GL va SaaS billing independent domains.

## Variantlar
1. Charge on create/draft: simple but canceled drafts may bill.
2. Charge on submit/confirm: human action; may duplicate later posting.
3. Charge on domain-post/finalize: more stable but not universal for all doc types.
4. Explicit versioned `BillableDocumentOccurrence` mapping per document type and transition: configurable and auditable, higher setup complexity.

## Tanlangan texnik taklif
**PROPOSED: option 4**, after user approves each document-type × transition mapping. No draft/confirm/post/return rule automatically APPROVED. Source domain emits immutable transition fact after server commit; Billing alone applies approved pricing, dedup, adjustment and reversal.

## Hujjat/transition candidate matrix (ALL OPEN except duplicate prevention proposal)

| Type | Transition | Candidate | Status |
|---|---|---|---|
| Sales order | draft/create/edit | no charge | OPEN |
| Sales order | submit/confirm | charge base + excess **or** defer | OPEN |
| Sales order | dispatch/close | no second charge if prior occurrence rated | OPEN |
| Purchase order | draft/approve | approve charge **or** no charge | OPEN |
| Goods receipt | inspected/posted | charge **or** no charge | OPEN |
| Stock transfer | dispatch/receive | once per logical transfer **or** per document | OPEN |
| Return | approve/post | charge, no-charge or original reversal | OPEN |
| Count/adjustment | posted | charge **or** no-charge | OPEN |
| Any document | amend/cancel/reverse | delta, refund, no refund or compensate | OPEN |
| Any document | retried operation/import | never double-charge same approved occurrence | PROPOSED invariant |

## Candidate contract
`tenant_id, company_id, branch_id?, document_type, source_document_id, source_revision, transition_name, transition_id, billable_occurrence_id, related_original_occurrence_id?, effective_at, server_committed_at, quantity, unit, schema_version, idempotency_key`. Unique billable occurrence distinct from transport event; correction links original charge and appends reversal/delta ledger entry. Transition does not imply stock/GL operation.

## Sabab
Avoids charging every state change, keeps document volume measured by correct owner and permits pricing evolution without silent retroactive charges.

## Xavflar
Misconfigured types, duplicated imports, partial fulfillments, refunds after finalized invoice, historical re-rating. Matrix and return/refund treatment require product/business approval.

## Boshqa modullarga ta'siri
[Sales & Inventory](../modules/SALES_INVENTORY.md) owns source workflow; [Mobile](../modules/MOBILE_ARCHITECTURE.md) syncs commands but never charges offline; Backend outbox and Integrations import identity. No modifications to those docs here.

## Approval gate
AR-010 and MR-04 remain OPEN pending business+Sales+Billing joint decision, including per-type eligibility, volume metric, cancellation/refund and integration behavior. No implementation authorization.

## Round 2 clarification — AR-010 / MR-04 (2026-10-12, PROPOSED)
Extended [type × transition × base/overage × cancellation/import matrix](../modules/BILLING_ARCHITECTURE.md#204-ar-010--mr-04--document-charging-and-volume-matrix). Sales Order confirm/finalize; Sales Invoice issue/post; Purchase Order approve; Goods Receipt post; Stock Transfer dispatch/complete; Return/Credit Note post; Inventory Adjustment post; imported external document accepted/finalized are **alternative OPEN charge triggers, not APPROVED**. For each separately decide base fee once per logical billable occurrence, source-authored volume/unit/threshold, correction delta vs reversal, partial fulfillment, cancellation policy and external legacy recognition. Semantic source_system+document+revision+transition identity distinct from transport event ID; recurring import must not double-charge. Sales/Finance stock/GL effects are separate.
