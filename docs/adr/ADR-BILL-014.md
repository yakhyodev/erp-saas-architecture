# ADR-BILL-014 — Chargeable Business Documents, Resource metering and progressive tier policy

- **Status:** `PROPOSED` — `DESIGN_SELECTED_UNDER_DELEGATION`, not published price/contract approval
- **Date:** 2026-10-12; **Owner:** 01 Billing; 05 Sales, 06 Finance, 09 Mobile, 12 Integration, 04 Backend
- **Related:** AR-009/010; MR-04/05; [BILL-009](ADR-BILL-009.md), [BILL-007](ADR-BILL-007.md), [BILL-011](ADR-BILL-011.md), [BILL-013](ADR-BILL-013.md)
- **Source:** user's delegated best-path architectural planning request dated 2026-10-12. Previously approved per-document base + over-threshold extra credit, marginal tiers candidate, 1 credit=10 UZS and actual-only charges remain binding.

## 1. Scope and evaluated alternatives
Alternative A: charge on create/draft (simple, unfair aborted drafts); B: charge canonical final/post transition (selected design default, high auditability); C: charge every transport event (reject, duplicate risk). Economic event created only when source owner has durably committed **eligible** domain transition; Billing alone decides actual charge under an active effective policy. ERP GL event/StockMovement != automatically SaaS charge.

## 2. MVP candidate type × economic occurrence

| Source document | Single candidate trigger | MVP charge policy status | Duplicate guard |
|---|---|---|---|
| **SalesOrder** | `CONFIRMED` once per stable logical order | Selected candidate; require published configured fee | Same logical order/revision transition, one base fee despite partial dispatch |
| **GoodsReceipt** | `POSTED` once per accepted receipt | Selected candidate; require published configured fee | Per canonical receipt, not each line/partial backend callback |
| **StockTransfer** | `COMPLETED` once per logical transfer | Selected candidate; require published configured fee | Dispatch/receive partial legs do not duplicate base |
| SalesInvoice generated from SalesOrder | No separate base in MVP unless explicit tariff enables independent eligible invoice | **Disabled by default** to avoid order+invoice double commercial charge | Maintain source lineage |
| PurchaseOrder | No base by default | Deferred, configurable later | Don't count approval and goods receipt twice without policy |
| Return/CreditNote/Correction | No independent base by default | Original-linked reversal/delta under policy; refund is **not automatically implied** by stock/accounting return | Stable original charge link |
| InventoryAdjustment | No base by default | Deferred | Stable count session/source revision |
| Imported historical documents | No historical billable event by default | Excluded absent explicit migration/new-post policy | Provider scoped external ID+revision+transition |
| Reposted/amended eligible document | Original-linked correction, not a fresh base by default | Versioned policy; positive delta only under approved definition | Business occurrence persisted |

**Critical distinction:** "selected default chargeable types" is NOT authorization to debit Wallet until commercial catalog has explicit published type+transition+price+measurement policy, effective dates and customer contract notification. Unconfigured category = `NOT_CHARGEABLE_PENDING_POLICY`; do not invent 0-cost-but-billed docs or a fee. Tenant/company scope and independent Wallet liability per ADR-BILL-011.

## 3. Definition of a chargeable occurrence
`tenant_id, source_company_id, source_branch_id?, document_type, canonical_document_id, logical_document_lineage_id, business_transition, source_revision, business_occurrence_id, transport_event_id, source_effective_at, server_accepted_at, policy_version, price_version, metric_name, quantity, unit, billing_account_id, charge_id, related_original_occurrence_id?, correction_reason?`.

- Product domain writer commits one semantic event via owner-local transaction+outbox. Billing resolves company/debtor from server trusted effective mapping and price eligibility, not from client field. Unique economic effect `(billing_account_id, business_occurrence_id, charge_component_version)` even if multiple event IDs/retries.
- A document may change state many times; selected transition occurs once per logical document unless an explicitly approved re-bill amendment policy applies. Same-key/different-payload -> conflict, no second charge.
- Partial shipment/partial goods receipt follow the correct *source document identity*, no repeated base per transport event; separately created genuinely distinct receipts need type policy.
- Cancellation before billable transition => no charge. Cancellation after charge => no automatic refund: documented adjustment policy, original linkage, immutable account ledger.
- Offline queued draft is NOT a committed billable event; only server-authorized accepted transition counts. External imports use provider namespace and stable canonical mapping; an imported historical doc cannot be recharged on migration retry.

## 4. Volume and progressive tiers — selected candidate, prices OPEN

1. Business Usage = document type-specific base credits plus additional credits for **actual measured metric strictly above the published type threshold**, using consistent explicit unit (e.g., eligible lines/quantity/size) and policy version. Type-specific metric, base amount, threshold and overage price **OPEN**, not guessed.
2. Per-company `billing_account_id` price/tier scope aligns with separate Wallet. `DESIGN_SELECTED`: calculate **marginal tiers** (each range uses its own rate), not retroactive rerating of the entire period. Tier aggregation window defaults to BillingAccount's named calendar billing month, but exact account billing timezone and settlement cutoff must be configured.
3. Effective price precedence candidate: explicit active EnterpriseContract final net terms for its permitted components; for other components, published standard base+overage rate then marginal progressive tier; **no automatic stacked double discount**. Contract overrides must declare component scope, priority and time validity. No invented discounts or quotas.
4. Usage period != posting period: trusted server-approved domain-effective month for economic classification; if already closed, immutable adjustment posted current open period referencing original. Historical ledger/pricing never updated. Device clock not authoritative; original tier recomputation effects require explicit signed accounting policy.
5. Resource Usage only for explicitly enabled, published, truly metered types (e.g., storage byte-days, API calls, processing units). Source unit+interval+dedup+provenance mandatory; unavailable source mapping => `PENDING_ATTRIBUTION`, not guessed payer. Company-scoped usage to owning Company; Tenant shared to explicitly accepted responsible Company from ADR-BILL-011. Storage meter periods crossing assignment boundary require verified segmentation or pending review.
6. Credits stored exact fixed precision; 1 credit=10 UZS; rounding policy version must be published before any non-integer charge. Each ledger economic effect append-only and reconcilable to meter/price/contract and source occurrence.

## 5. Invoice, reconciliation and warnings

SaaS Billing statement/invoice is never tenant ERP SalesInvoice/Finance GL. Draft estimate is not posted balance. UI must show Billable candidate / Meter pending / Rate pending / Charge posted / Adjustment / Payment confirmed separately, with branch/company filters and effective policy links. Cross-company offset prohibited; authorized third-party payer funding only after provider settlement verified. Chargeback/partial refund/dispute and tax/fiscal invoice specific rules `OPEN` pending provider/jurisdiction evidence.

## 6. Test contract (design only, NOT EXECUTED)
- Duplicate `CONFIRMED` and `COMPLETED` events (new event_id same business occurrence) => one base charge; changed payload => conflict.
- SalesOrder CONFIRMED→partial DISPATCH→INVOICE => only first configured SalesOrder charge; no invoice double-charge by default.
- Return and correction tied to original, no automatic credit beyond approved policy.
- Purchase/import historical legacy docs => zero *new* charge by default, still reported as historical.
- Tier boundary =/above, marginal calculation and enterprise component priority; no rounding drift/no rate without published tariff.
- Cross-tenant/branch forged payer Wallet, multi-Company, shared resource reassignment => no source-to-Wallet misrouting.
- Offline replay late month + closed invoice => immutable linked accounting adjustment, no duplicate billing.
- Unconfigured monetary tariff => no Wallet debit, safe pending/not-chargeable result; NO estimated ceiling.

## 7. Risks and undecided conditions

Scope completeness vs revenue; intentionally limited initial chargeable types can be expanded by separate approved price catalog, not by generating hidden fees. Source semantic IDs and domain trigger accuracy essential; source transition must not mutate inventory or Finance GL incorrectly. **OPEN:** numeric fees, overage thresholds, resource types/rates/free included amounts, enterprise written offers, tax compliance, refund/cancellation policy, time-zone effective dates, seat monthly rate, service grace after expiry. No implementation/production tests were executed. This ADR is not blanket economic APPROVED.
