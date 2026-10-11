# BILLING_ARCHITECTURE.md

> ERP SaaS Billing & Monetization · 2026-10-11 · v0.1 · Document status **PROPOSED**
>
> **APPROVED** below refers only to user-confirmed business rules. Technical design remains **PROPOSED**. No code.

## 1. Approved business constraints

- **APPROVED:** Hybrid Prepaid SaaS (wallet top-up and actual usage billing).
- **APPROVED:** Low-cost Seat + Business Usage + Resource Usage.
- **APPROVED:** Business Usage: base charge per document, plus credits after the volume threshold.
- **APPROVED:** 1 credit = 10 UZS.
- **APPROVED:** Progressive volume discounts; individualized enterprise contracts.
- **APPROVED:** When balance is exhausted, **actual** Seat and Usage charges are recorded as negative balance for **one calendar month**. **No estimated credit limit**.
- **APPROVED:** Companies and branches managed on one SaaS platform.
- **OPEN:** Exact price points, volume metrics, negative-period anchoring and post-expiry service policy.

## 2. Module boundaries

Billing owns wallet/ledger, metering, rating, pricing version, billing invoice, top-up posting, reconciliation, and billing service-status output. Tenant Management owns tenant/branch structure; IAM owns account/membership lifecycle; Sales/Inventory owns document validity; Storage owns measured consumption; Payments provider owns settlement; ERP Finance owns customer-facing books. SaaS billing invoices are **not** ERP sales invoices.

## 3. Wallet — PROPOSED

One billing_account and wallet per tenant company, branch attribution per entry. Balance is ledger-derived; cached balance is a transactionally consistent projection, never the only source of truth. Charges in integer or fixed-precision credits, never floating-point. Confirmed top-up increases balance; rated actual usage decreases it. Negative balances are valid during approved calendar-month period; never compute a forecast or estimated overdraft ceiling. Concurrent posting requires atomic transactions, unique keys and consistency checks.

## 4. Credit ledger — PROPOSED

Append-only double-entry journal with immutable posted ledger_transaction and ledger_entry. A correction is a linked reversal and new entry, not an UPDATE/DELETE. Store tenant_id, branch_id if applicable, event_id, charge_id, pricing_version_id, actor, correlation_id, occurred_at and posted_at. Enforce balanced debits/credits per transaction, unique posting reference, tenant isolation and full audit. Recompute/reconcile projections periodically; Finance business GL remains separate.

## 5. Usage event architecture — PROPOSED

Producers emit events after business commit via transactional outbox or equivalent. Contract: event_id, tenant_id, branch_id?, event_type, source_document/resource_id, source_version, quantity, unit, occurred_at, producer, schema_version, correlation_id. Consumer supports at-least-once delivery and exactly-once **economic posting** by durable dedup key. Stages: received, validated, rated, posted; transient retry, dead-letter, rejected or conflict states. Same event ID with changed payload is flagged, not double-billed. Document edits/cancellations, late/offline events and compensations need explicit policy.

## 6. Seat billing — PROPOSED

Use IAM membership history and effective-dated seat snapshots. Billable status, monthly/daily basis, prorating, invitation/suspension semantics and rates are **OPEN**. Seat charge references tenant, member/snapshot, pricing version and period. Replayed membership events cannot double-charge.

## 7. Business Usage — PROPOSED

For each billable document: `base_document_credits + excess_volume_credits(volume, threshold, rate)`, then apply approved discount contract/version. Version the metric, threshold, unit, rounding rule and billable document lifecycle. Record each rating component. Do not charge drafts/revisions/returns absent explicit policy; reversal/compensating event for approved refunds, never remove history.

## 8. Resource Usage — PROPOSED

Meter eligible storage, attachments, API/compute and other resources **only when product owners approve billable categories**. Producer measurements must be attributable to tenant, metric, unit, time window and source; deduplicate overlapping samples. Aggregation, free inclusion, units and prices remain OPEN.

## 9. Pricing & enterprise contracts — PROPOSED

Immutable effective-dated pricing_version and price_rule records. Historical charge references exact pricing version and formula components; never silently re-rate posted transactions. Enterprise contract stores effective interval, tenant, overrides and approval history. Precedence between base price, enterprise override, discounts, tax and rounding is OPEN and must be deterministic when approved.

## 10. Progressive discount engine — PROPOSED

Progressive volume discount is APPROVED; size brackets, percentage, time window, applicability and stacking are OPEN. Marginal tiers are recommended for predictability but not approved (ADR-BILL-007). Store tier-by-tier calculation breakdown to enable audit and explainability.

## 11. Billing invoice — PROPOSED

billing_invoice, invoice_line, billing_statement, payment_allocation and adjustment/credit_note. Itemize Seat, Business Usage, Resource Usage, discounts, authorized tax and reversals; every line references posted charges. Final invoice immutable; adjustments are additional documents. Invoice numbering, taxation, legal compliance, due dates and payment terms OPEN. Never confuse SaaS invoice with ERP customer's sales invoice.

## 12. Payment reconciliation — PROPOSED

Receive verified provider callback; do **not** credit wallet on initiation or unverified webhook. Confirm settlement/reference/amount and uniqueness before immutable deposit posting. Support pending, confirmed, failed, disputed, refunded and reconciled state as appropriate. Webhook retry is idempotent; daily/provider statement reconciliation identifies missing, excess, duplicate, reversed or disputed transfers; exception cases go to controlled workflow.

## 13. One-calendar-month negative balance — APPROVED constraint, OPEN lifecycle details

The real Seat and Usage charges continue as negative posted ledger entries for one calendar month, with **no forecast credit limit**. Specific grace start/end anchor, timezone, partial refill, repeated negative cycles, delayed usage, expiry notification, post-expiry feature limits and recovery remain **OPEN**. A proposed service-policy state machine must not silently block service without explicit approval.

## 14. Security, audit and performance — PROPOSED

Every change to prices, contracts, reconciliation, manual adjustments and billing status must have actor/time/reason/correlation. Strict tenant isolation and least privilege; no cross-company reads or writes. Signed webhook verification, secret management and no raw payment card storage. Tenant/time/source compound indexes; optional time partition of large ledger and usage events; async read models; retry and observability with trace IDs. Evaluate transaction serialization under high concurrency. Go, PostgreSQL and modular monolith remain technology proposals.

## 15. ADR index

- [ADR-BILL-001 Tenant-level wallet](../adr/ADR-BILL-001.md)
- [ADR-BILL-002 Immutable ledger](../adr/ADR-BILL-002.md)
- [ADR-BILL-003 Usage-event idempotency](../adr/ADR-BILL-003.md)
- [ADR-BILL-004 Pricing versions](../adr/ADR-BILL-004.md)
- [ADR-BILL-005 Payment reconciliation](../adr/ADR-BILL-005.md)
- [ADR-BILL-006 Negative-balance lifecycle](../adr/ADR-BILL-006.md)
- [ADR-BILL-007 Progressive discount tiers](../adr/ADR-BILL-007.md)
- [ADR-BILL-008 Invoice immutability](../adr/ADR-BILL-008.md)

## 16. OPEN_QUESTIONS proposals (not approved)

Q01 Seat price and charge cycle; Q02 document category/base price; Q03 volume metrics and thresholds; Q04 resource types and unit costs; Q05 discount thresholds and stacking; Q06 calendar-month start/end and timezone; Q07 post-expiry service policy; Q08 cancellation/edit/return charge reversal; Q09 seat proration and statuses; Q10 refund/chargeback; Q11 branch cost allocation; Q12 enterprise override precedence; Q13 taxes/legal invoice requirements; Q14 payment providers/settlement; Q15 offline/late event handling; Q16 currencies and rounding; Q17 recurring negative-period reopening.

## 17. Codex implementation specification — no code

Domain models: BillingAccount, Wallet, LedgerTransaction, LedgerEntry, UsageEvent, Charge, SeatSnapshot, ResourceMeasurement, PricingVersion, PriceRule, DiscountTier, EnterpriseContract, BillingInvoice, InvoiceLine, Payment, ReconciliationCase, NegativeBalancePeriod, AuditRecord.

Cross-module event/API contracts: Tenant membership and branch, IAM seat lifecycle, document finalization/correction, resource measurement, payment settlement verification, notification status. Each write carries tenant_id, idempotency key, correlation_id and version.

Acceptance criteria: ledger debits equal credits; posted entries immutable; replay does not double-charge; settlement required for top-up; historical pricing reproducible; cross-tenant access rejected; concurrent posting keeps wallet and ledger consistent; negative period reflects **real** charges and has no estimated cap; no unapproved post-expiry shutdown rule. Implement neither pricing values nor edge-case policy without approval.

## 18. Shared-document change proposals (not applied)

DECISIONS.md: reference the approved billing business rules, but list ADR-BILL-001..008 as PROPOSED. MASTER_ARCHITECTURE.md: describe Billing boundary and Tenant/Finance/IAM contracts. REQUIREMENTS.md: add ledger, audit, idempotency and approved negative-balance invariants. MODULE_REGISTRY.md: add billing dependencies. OPEN_QUESTIONS.md: add Q01–Q17. CHANGELOG.md: note creation of module architecture proposal. Do not replace these shared files without explicit synchronization review.

---

## 19. Cross-module harmonization (AR-001, AR-009, AR-010; MR-02…MR-05) — 2026-10-11

> **Status: PROPOSED CLARIFICATION / OPEN DECISIONS.** Audit source: [historical architecture review](../reviews/ARCHITECTURE_REVIEW.md); cross-module registries: [DECISIONS](../../DECISIONS.md), [REQUIREMENTS](../../REQUIREMENTS.md), [MODULE_REGISTRY](../../MODULE_REGISTRY.md), [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md). Historical audit is not current approval. Existing sections 3, 5, 6, 7, 9, 10, 13 must be read with this section: earlier “one wallet per tenant company” is **only option B**, not fixed cardinality.

### 19.1 AR-001 / MR-01 — Tenant → Company → BillingAccount → Wallet

**Ownership, not asserted cardinality:** Tenant Registry is authoritative for tenant/company/branch/legal entity identifiers and lifecycle; Billing is authoritative for BillingAccount, payment responsibility mapping, Wallet and SaaS debt/ledger; Finance is authoritative for ERP GL/legal accounting. A branch belongs to the tenant/company relationship validated by Multi-Tenant; no `billing_account_id = tenant_id` or `company_id = tenant_id` assumption. LegalEntity identity and payer relationships remain OPEN.

| Variant | Tenant → Company | Company → BillingAccount | BillingAccount → Wallet | Payer / debt holder | Trade-off / status |
|---|---|---|---|---|---|
| A. Tenant consolidated | 1:N candidate | N:1 shared tenant account | 1:1 candidate | Tenant-contracted payer | Simple consolidated billing; legal company debt allocation complex; PROPOSED |
| **B. Company account (preferred candidate)** | 1:N candidate | 1:1 per billable company | 1:1 candidate | Company-contracted payer/legal entity TBD | Cleaner attribution, more top-ups; **PROPOSED**, not APPROVED |
| C. Flexible contract accounts | 1:N candidate | N:M via effective-dated company/account assignment (explicit mapping) | 1:N candidate | Contract payer/account debt holder | Enterprise flexibility, complex invoices/migration/reconciliation; PROPOSED |
| D. Branch wallets | 1:N company; 1:N branch | 1:N branch accounts | 1:1 or 1:N | Branch payer only if lawful/contractual | Maximum separation, operational burden; PROPOSED, not default |

**Required contract (PROPOSED):** `tenant_id`, `company_id`, optional `branch_id`, explicit `billing_account_id`, `wallet_id`, `payer_party_id?`, `legal_entity_id?`, `assignment_version`, `effective_from`, `effective_to`. Each usage/seat charge carries source tenant/company/branch/user-or-membership, source document/operation, producer identity, rating version, charge period, `ledger_transaction_id` and immutable assignment snapshot. Server validates account-to-tenant/company membership at event effective time; no client-supplied wallet authority. Ledger entry must separately retain **source attribution** (where activity happened) and **liability attribution** (which account owes), including reassignment/correction linkage; do not mutate old entries when branch/account moves. Scope, payer, enterprise multi-company consolidation, wallet pooling, tenant migration, restore and negative-debt transfer stay OPEN. Even with single wallet, branch analytics do not create separate debt unless approved.

**Cross-module handoff:** compatible with [Multi-Tenant §10](MULTI_TENANT_ARCHITECTURE.md) optional company/branch and independent BillingAccount mapping; IAM controls memberships, not financial payer; enterprise isolation/sharding follows immutable tenant context. Do not merge SaaS credit ledger with [Finance accounting](FINANCE_ARCHITECTURE.md) or presume FIN-001 provenance approved.

### 19.2 AR-010 / MR-04 — Document type × transition × charge policy

Sales/Inventory owns authoritative document state, type, revision and legal/stock consequences; Billing owns price, charge eligibility, dedup and reversal. **No lifecycle row below is APPROVED.** “No charge” is a **suggested safe default**, not a user-approved exclusion. Transition facts are not directly interchangeable with billable event: derive a canonical billable fact only under a versioned approved mapping.

| Source document type | Transition / event | Candidate action | Decision status / caveat |
|---|---|---|---|
| Sales order | create / edit draft | No charge candidate | OPEN: draft documents could become billable |
| Sales order | submit | Charge **or** no charge | OPEN: submitted vs confirmed vs finalized boundary |
| Sales order | confirm | One base charge + excess volume **or** defer | OPEN: candidate charge trigger; no implicit approval |
| Sales order | dispatch / partial fulfilment / close | No second base charge if already charged; otherwise evaluate | OPEN: physical stock posting is not automatically billing posting |
| Purchase order | draft → approved | Charge **or** no charge | OPEN: which purchased documents billable |
| Goods receipt | inspected → posted | Charge **or** no charge | OPEN: receipt may be billable independently |
| Stock transfer | dispatch / receive / partial receive | Per logical transfer **or** per separately approved posted document; no duplicate per leg | OPEN: aggregation and policy |
| Customer/supplier return | return requested / approved / posted | No charge, new charge **or** reversal/credit against original | OPEN: returns not always refunds |
| Inventory adjustment/count | approve / post | Charge **or** no charge | OPEN: scope/unit |
| Any previously charged document | amend / correction / revise | Delta charge **or** reverse + re-rate; never duplicate original | OPEN: pricing effective date and accounting-period correction |
| Any charged document | cancel before/after final posting | No new charge; original reversal **or** nonrefundable charge | OPEN: cancellation/refund policy |
| Any type | retry / duplicate import / offline replay | Never second economic charge for same business occurrence | PROPOSED idempotency safety invariant; charge eligibility remains OPEN |

**PROPOSED canonical contract:** `tenant_id, company_id, branch_id?, document_type, source_document_id, source_revision, transition_name, transition_id, billable_occurrence_id, related_original_occurrence_id?, source_effective_at, server_committed_at, quantities, volume_unit, actor_id?, schema_version`. Include `rating_policy_version`, `idempotency_key` and `correlation_id` in Billing derived fact. Dedup **semantic business occurrence** separate from transport event ID; source revision plus canonical occurrence version must identify amendments. Same id/key but different content is conflict, never silently replace. Reversal references `original_charge_id`, carries its own unique occurrence, creates compensating ledger entries and invoice adjustment when finalized; no history deletion. Sales stock/Finance GL transitions do not automatically determine SaaS charging. Requires joint approval with [Sales & Inventory §10](SALES_INVENTORY.md), and integrations for import/external sources.

### 19.3 AR-009 / MR-02 — Offline, late usage, event times & billing periods

**Five distinct timestamps (PROPOSED):**

| Field | Meaning / authority |
|---|---|
| `client_occurred_at` | Device-observed action time; untrusted clock with offset/clock skew metadata |
| `server_committed_at` / `effective_at` | Authoritative backend business transition acceptance; can be later than device action; source-domain controlled |
| `received_at` | First durable receipt by backend/Billing inbox |
| `rated_at` | Pricing version applied and rated charge computed |
| `posted_at` | Immutable ledger commit time; must never be backdated |
| `billing_period_key` | Separately stored attributed period selected by approved period policy (not automatically `posted_at`) |

Candidate month policies (all **OPEN**): **A** server-effective month, potentially reopen prior month with controlled adjustment; **B** first receipt month, easier close but deviates from economic occurrence; **C** device occurred month only after server validation, with bounded accepted clock skew (limits OPEN); **D** hybrid: pre-close effective month, post-close current-month correcting adjustment referencing original. **Preferred review candidate: D with server-authoritative effective timestamp**; not APPROVED. Store `original_period_key`, `posting_period_key`, `adjustment_reason`, `source_event_id` and `original_charge_id?` to prevent silent retrospective edits. Month close, invoice finalization, late cutoff, discount accumulation and version selection OPEN. Never silently re-open immutable finalized invoice.

**End-of-month example (illustrative, not policy):** device creates operation at local 31st 23:58; sync received 1st 09:00; server accepts transition 1st 09:01; pricing 1st 09:02; ledger posts 1st 09:03. Device time, authoritative business time, ingestion month and posting month differ. Under option A/B/C/D, liability and discount tiers can differ; choose only after Mobile/Sales/Backend sign-off. Offline device does not charge, reserve stock or post to SaaS ledger; server acceptance is a precondition. If server rejects command, no economic event. This aligns with [Mobile Stock and Billing](MOBILE_ARCHITECTURE.md) and [ADR-MOB-008](../adr/ADR-MOB-008.md).

### 19.4 MR-02 — One calendar month negative-balance timeline

**Only APPROVED invariant:** when balance is exhausted, actual Seat/Business Usage/Resource Usage debits may continue negative for **one calendar month**; **never** estimate a debt ceiling / remaining credit cap. A **calendar month** is not silently equated to rolling 30 days.

| Timeline step | Data/state candidate | Open business decision |
|---|---|---|
| T0 balance reaches zero or below | Ledger records actual debit and `first_negative_at` | Trigger on crossing zero, at 0 exactly, or first negative; whether partial previous debt carries |
| T1 month window assigned | `negative_period_id`, `calendar_month_key`, timezone policy version | Start of current month to month-end **vs** next full calendar month **vs** one calendar-month anniversary interval; what “bir kalendar oy” means |
| T2 subsequent real usage/Seat | Continue posted actual charges; negative value derived, **no forecast ceiling** | Late events belonging before/after window, seat accrual frequency |
| T3 refill / positive recovery | Verified top-up journal entry and reconciled balance | Does refill close/reset window? What if partial top-up leaves balance negative? |
| T4 expiry | `expiry_at`, action pending | Notification, permitted read/write actions, grace, collections, whether to suspend; **not approved** |
| T5 later recovery / repeat crossing | Linked successor periods and historical audit | New window eligibility, unpaid past-period charges, delayed adjustments |

**Candidate interpretations:** month-to-month anniversary with calendar arithmetic; current named calendar month until its last instant; next named whole calendar month. No variant may silently become APPROVED. Persist timezone ID, `period_policy_version`, `period_start_at`, `period_end_at` using half-open intervals; server converts to UTC instants. Example: an October 28 deficit and a November 2 delayed event require separate event-attribution and grace-window decisions; **neither** forecast ceiling nor arbitrary automatic shutdown allowed. Wallet balance, grace eligibility and legal invoice due date are distinct concepts. Delay, renewal, top-up and expiry require joint business approval before service-enforcement code.

### 19.5 MR-03 — Seat activation, prorating and identity

**PROPOSED source contract:** IAM/Identity emits membership events (`invited`, `accepted`, `activated`, `suspended`, `reactivated`, `removed`) with `tenant_id`, `company_id?`, `membership_id`, `user_id`, `role/scope?`, `effective_at`, `transition_id`, `source_version`; Billing computes `billable_seat_snapshot` by approved statuses and effective dates. Account/branch liability uses AR-001 mapping snapshot, not a guessed tenant mapping. Membership may span branches: avoid counting same purchased seat twice without explicit plan policy.

| Seat scenario | Option A | Option B | Status |
|---|---|---|---|
| Invite pending | Not billable until acceptance/activation | Billable at invitation | OPEN |
| Activation | Bill from activation (prorated) | First full billing interval | OPEN |
| Suspension | Stop accrual at suspension | Still bill reserved seat | OPEN |
| Mid-period join/leave | Calendar-day prorate | Instant-based prorate or full interval | OPEN |
| Multiple branches/devices | One company/tenant membership seat | Distinct scoped seats | OPEN |
| Seat discounts | Pricing tier after measured active seats | Enterprise contract seat override before discount | OPEN |

**Idempotency proposal:** unique `(tenant_id, membership_id, transition_id)` for state input; unique `(billing_account_id, membership_id, seat_period_key, charge_component_version)` for periodic economic charge (or interval ID for prorating). Replayed `activated` transition must not reset `effective_at` or add duplicate bill. Corrections reference original accrual/charge and preserve prior posting. Proration fractions use fixed-precision, explicit timezone, leap/month boundary handling and deterministic rounding **after** pricing policy approval. Billing period and discount tier base must be independently approved. Identity remains membership source of truth; Billing owns rating, not IAM role policy.

### 19.6 MR-04 / MR-05 — Volume, resource, discount and pricing conflict gates

**Approved:** document base price + extra charge above threshold; progressive discounts; enterprise individualized pricing. **OPEN:** billable document types/transitions, row/count/bytes/units measure, threshold inclusive/exclusive boundary, volume measurement on update, resource resource_type/units, zero/free allowance, rounding credit granularity, progressive tiers and measuring time window.

**PROPOSED pricing evaluation graph (subject to contract approval):** validate authenticated tenant/company & event → resolve legal/payer account assignment snapshot → resolve document/resource/seat eligibility → select effective immutable `pricing_version` based on chosen billable date → calculate base and over-threshold components → choose contract override precedence → calculate progressive discount per approved tier algorithm → round deterministically → record full breakdown & post immutable ledger.

| Discount/enterprise alternatives | Advantages | Risks / OPEN |
|---|---|---|
| Marginal progressive tiers | Gradual incremental tier price | Tier unit and scope/cumulative window OPEN |
| Retroactive whole-volume tiers | Easy explanation in some tariffs | Threshold discontinuities and month-end re-rate OPEN |
| Enterprise override replaces base tariff, then tier discount | Predictable custom contract price | Stacking may over-discount; precedence OPEN |
| Enterprise fixed net price, no standard discount | Easier enterprise audit | May conflict with negotiated progressive benefits; OPEN |
| Multiple contracts per billing account | Flexible | Overlap and priority need deterministic tie-breakers |

Pricing snapshots must record `pricing_version_id`, `contract_id/version?`, `discount_policy_version`, `tier_calculation`, `measurement_source`, `rounding_policy`, `billing_period_key`, `charge_origin`. Historical posted entries never auto re-price, including offline arrivals; if adjustment policy approved, use linked immutable correction. **Do not invent numeric prices, thresholds, resource unit rates, prorations or effective dates.**

### 19.7 Cross-domain approval gates and tests — NOT IMPLEMENTATION AUTHORIZATION

- **AR-001:** Multi-Tenant + Billing + Finance approval for entity/account/legal payer and branch transfer. Test cross-company leakage and migration with debt/credits separate.
- **AR-009:** Mobile + Sales/Backend + Billing approval for timestamps, skew, late-month cutoff and immutable adjustment. Test offline October→November replay, reversed operation, two devices same command, failed server acceptance, finalized invoice.
- **AR-010:** Sales/Inventory + Billing + Integrations approve document type × transition matrix and correcting/reversal rules. Test confirm/dispatch/return duplication, cancellation after invoice, amended volume, imported documents.
- **MR-02:** Business approval of exact one-calendar-month anchor, timezone, refill, expiry/recovery. Test boundary, partial refill, leap/month-length differences, non-authorized shutdown; no forecast credit limit.
- **MR-03:** Identity + Billing + Business approve billable statuses, prorating cadence, seat scope, discounts. Test invite→activate→suspend→reactivate; simultaneous branches and duplicate events.
- **MR-04/MR-05:** Business + Billing approve thresholds, resource dimensions, effective pricing/discount precedence, rounding. Test threshold crossing, tiers, enterprise overlaps, corrections after pricing change.
- A user-facing UX should display pending sync, disputed/reversed charges, period distinction and balances understandably in **uz/en/ru**, while hidden future alphabet option remains supported. Usability is APPROVED; exact UI is PROPOSED.

**Local ADR additions:** [ADR-BILL-009](../adr/ADR-BILL-009.md) (document charge boundary) and [ADR-BILL-010](../adr/ADR-BILL-010.md) (seat lifecycle/proration). ADR-BILL-001/003/004/006/007 are clarified to reference this section. All remain **PROPOSED** until user-approved.
