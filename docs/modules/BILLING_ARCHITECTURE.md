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

---

## 20. Round 2 correction — AR-001/009/010/013, MR-01…05 (2026-10-12)

**STATUS: PROPOSED architecture + OPEN business decision packet.** This section is more recent than earlier candidate wording in §§2–3, 6 and 19. Earlier “IAM owns membership” and “one wallet per company” are not approved facts. Source: [Round 2 audit](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md), [Multi-Tenant](MULTI_TENANT_ARCHITECTURE.md), [Finance](FINANCE_ARCHITECTURE.md), [Sales](SALES_INVENTORY.md), [Mobile](MOBILE_ARCHITECTURE.md), [Security](SECURITY_ARCHITECTURE.md); root [DECISIONS](../../DECISIONS.md) and [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) remain unmodified.

### 20.1 AR-001 / MR-01 — Legal payer, debt owner and cardinality

**APPROVED:** many companies/branches within one SaaS. **NOT APPROVED:** one tenant equals one legal company, one billing account per company, or any specific payer cardinality. Separate organizational ownership, contracting payer, beneficial beneficiary, settlement source and ledger liability.

| Candidate | Tenant : Company | Company : LegalEntity | Company : BillingAccount | BillingAccount : Wallet | Payer/legal liability | Advantages | Risks |
|---|---|---|---|---|---|---|---|
| A — consolidated tenant | 1:N | N:1 or 1:N, OPEN | N:1 | 1:1 | Contracting LegalEntity linked to tenant account | Few balances/payments | Intercompany debtor/tax/invoice allocation |
| **B — company-based MVP (recommended PROPOSED)** | 1:N | 1:1 candidate with effective dates | 1:1 candidate | 1:1 candidate | Explicit LegalEntity payer and debt holder per account; third-party payer separate | Predictable ledger, simpler isolation/reconciliation | Shared legal entity/multiple subsidiaries need consolidation |
| C — contract pooling | 1:N | configurable | N:M with effective-dated allocation, no implicit cross-company charge | 1:N or 1:1 by wallet purpose | Contract payer may fund multiple companies, debt assignment explicit | Enterprise flexibility | Harder permission/debt migration, statement and refunds |
| D — per-branch | 1:N | company-defined | company: N branch accounts | 1:1 per account | Payer defined by contract, not guessed from branch | Isolated budgets | High operational overhead and duplication |

**Recommendation B** only for MVP simplicity, not new approval. Do not treat legal-entity mapping as necessarily 1:1: confirm company↔LegalEntity and third-party payer cardinality before migration. Proposed immutable ledger attribution: `tenant_id, source_company_id, source_branch_id?, source_membership_id?, billing_account_id, wallet_id, legal_debtor_entity_id?, payer_party_id?, payer_authorization_id?, mapping_version, source_fact_id, charge_id, priced_at, posting_period, original_transaction_id?`. `payer_party_id` identifies funder; **payer != legal debtor** by default. Positive top-up, real charge, refund, transfer or assignment change preserve independent postings and audit. Reassigning company to account cannot rewrite historical debt; migration requires approved settlement/transfer process. Tenant registry validates source company/branch; Finance validates legal entity and fiscal attributes; Billing decides rating and debt only under approved mapping. Reject ambiguous mapping, do not fall back to arbitrary tenant wallet.

### 20.2 MR-02 — Two interpretations of “one calendar month” (both OPEN)

**APPROVED invariant:** actual Seat + Business/Resource Usage charges can post to negative wallet balance for **one calendar month**, never a speculative estimated credit limit. This sentence does not determine the start/end rule.

| Policy candidate | Example first negative: 2026-10-28 16:00 Asia/Tashkent | Example first negative: 2026-01-31 10:00 Asia/Tashkent | Benefit / hazard |
|---|---|---|---|
| **N. Named calendar month** | Grace only to **Nov 1 00:00** if current October month is chosen (3+ days); alternative next named month would end **Dec 1 00:00** | To **Feb 1 00:00** if current January month (less than 1 day); alternative next named month to **Mar 1 00:00** | Fits reporting calendar; highly unequal grace depending on crossing date; current vs next month must be decided |
| **A. Anniversary-based calendar month** | End **Nov 28 16:00** (one calendar-month addition) | Feb has no day 31: clamp to **Feb 28 10:00** vs alternative rollforward to March (OPEN) | More consistent length from deficit; date arithmetic/DST and end-of-month ambiguity |

Examples are **illustrative and not approved timelines**. Rolling **30 days** is a different rule and cannot be substituted for calendar month. Business owner must choose N-current, N-next, A-clamp, A-rollforward or a specified alternative. Proposed period record: `first_negative_event_id, first_negative_posted_at, tenant_timezone_iana, negative_period_policy_version, start_at_utc, end_at_utc_exclusive, billing_account_id, recovered_at?, expiry_decision?`; preserve local wall-clock explanation and immutable UTC instants. Tenant timezone vs payer/legal entity timezone is OPEN. Window computation must use selected policy at creation and not change silently after timezone configuration change.

| Boundary situation | Proposed recorded behavior | OPEN approval choice |
|---|---|---|
| **First negative** | Detect posted real debit crossing previous balance from positive/zero into negative, store exact ledger event ID | At exactly zero trigger? multiple simultaneous debits serialization? |
| **Top-up** | Verified settlement posts new ledger credit, no predicted credit | Fully positive closes grace vs retain old window; partial top-up does not erase debt history |
| **Expiry** | Persist expiry state/event without silent service cut-off | Allow read, block new writes, contractual collections or another policy |
| **Repeat** | Preserve old negative period history and lineage | New window on repeat deficit vs retain original window; anti-abuse rule |
| **Closed month** | Never edit immutable prior invoices/entries; attach adjustment reference | Reopen accounting period vs current adjustment; notify debtor, tax implications |
| **Late usage** | Charge only validated business occurrence, linked original/effective/posting period | Assign charge and discount/negative grace to occurrence, receipt, or adjustment period |
| **Crossed period** | Separate invoice period, usage attribution period and negative grace period | Which clock controls eligibility for charge in grace after delayed event |

**Hard constraints:** no forecast ceiling, no silent estimated usage/debt-limit, no automatic post-expiry suspension until approval, no offline wallet debit and no mutation of posted ledger history.

### 20.3 AR-013 / MR-03 — Seat source of truth, lifecycle and ownership

**Single-writer candidate (PROPOSED, awaiting AR-013 approval):** **Identity** owns global `UserCredentials` and authentication; **Multi-Tenant** owns and writes `TenantMembership` lifecycle/status and company/branch affiliations; **Security** owns `RoleAssignment`, permission evaluation, grants and revocation enforcement; **Billing** consumes authorized, versioned membership facts and owns billable-seat snapshots, charge rules and posting. No duplicate membership write in Security and no seat charge based on a stale role/token. [Security §17](SECURITY_ARCHITECTURE.md), [Multi-Tenant](MULTI_TENANT_ARCHITECTURE.md) propose this but [MODULE_REGISTRY](../../MODULE_REGISTRY.md) still attributes membership to Security; **this registry mismatch remains OPEN**, not silently resolved here.

| Membership transition | Source event | Billing status options (all OPEN) | Risk control candidate |
|---|---|---|---|
| invited/pending | `MembershipInvited` | billable at invite **or** never until activation | stable membership ID, no duplicate invite charges |
| activated/accepted | `MembershipActivated` | full cycle **or** prorate from effective date | authoritative version and activation timestamp |
| suspended | `MembershipSuspended` | still reserved/billable **or** pause at effective date | distinguish auth suspension from commercial licensed seat |
| reactivated | `MembershipReactivated` | resume same seat or new seat interval | dedup and historical interval closure |
| removed/revoked | `MembershipRevoked` | immediate stop or end-of-cycle charge | authorization revoke immediate; pricing effect OPEN |
| company transfer | `MembershipCompanyScopeChanged` | liability change at effective date or next cycle | same tenant seat may be reallocated without duplicate company charge |
| branch/device overlap | `ScopeChanged/DeviceSession` | one membership seat or separately scoped license | do not count devices/roles as seats unless approved |

Proposed event envelope: `tenant_id, user_id, membership_id, membership_version, transition_id, status_before/after, effective_at, committed_at, company_scope_before/after, branch_scope, correlation_id`; Billing resolves approved BillingAccount mapping at charge-effective date. **Do not infer user↔membership cardinality equals chargeable seats.** Proposed idempotency: unique membership transition per `(tenant_id, membership_id, transition_id)`, unique economic accrual per `(billing_account_id, membership_id, seat_license_scope, accrual_interval, charge_component_version)`; stable changes have corrections not duplicate debit. Proration alternatives: none/full-month; calendar-day fraction; exact-time fraction. Invited, suspended, company-move, refund, cycle, rounding and seat tiers remain OPEN.

### 20.4 AR-010 / MR-04 — Document charging and volume matrix

**APPROVED:** base price per billable document + additional credits only beyond approved volume threshold. **All document types, transitions, fees, thresholds, refunds and import recognition below are candidate policy, not approved.** Suggested safe operational separation: one logical chargeable occurrence, not one arbitrary transport message.

| Type | Candidate billable transition | Base fee | Excess-volume credit | Cancel / return | Import / correction |
|---|---|---|---|---|---|
| Sales Order | CONFIRMED **or** FINALIZED | once per approved occurrence, amount OPEN | line/quantity/amount units OPEN | cancel pre/post charge: no-charge/refund/retain OPEN | imported confirmed doc treated as new **or** historical (OPEN); semantic source dedup |
| Sales Invoice (ERP) | ISSUED/POSTED **or** none | eligible? OPEN | line/taxable amount? OPEN | credit note vs refund policy OPEN | identify external origin and prior billed ID |
| Purchase Order | APPROVED **or** none | eligible? OPEN | line/quantity? OPEN | cancel/reopen OPEN | map imported PO status and original ID |
| Goods Receipt | POSTED **or** none | eligible? OPEN | receipt lines/units? OPEN | receipt reversal separate from billing credit | imported posted receipt mapping OPEN |
| Stock Transfer | DISPATCHED **or** COMPLETED **or** none | once per logical transfer vs per leg OPEN | transfer lines/qty? OPEN | partial receive/cancel/return OPEN | distinct source transition IDs by leg |
| Return / Credit Note | POSTED **or** none | independent document fee? OPEN | return lines/units? OPEN | original charge reversal or independent charge OPEN | `original_charge_id` linkage |
| Inventory Adjustment | POSTED **or** none | eligible? OPEN | adjusted lines/units? OPEN | corrective count/reversal OPEN | stable count session & import ID |
| Imported external doc | authoritative accepted transition **or** excluded historical | new fee only if approved, no implicit repeat | original or normalized metric? OPEN | imported cancel/refund mapped to local correction | `source_system + external_document_id + revision + transition` dedup |

Each charge policy version must specify type, transition, eligibility, base-credits schedule, measured volume unit, threshold inclusive/exclusive, overage rounding, full vs delta on amendment, partial flow, cancellation/reversal rules and source-origin mapping. **Draft/create, confirm, dispatch and GL posting are not automatically billable**; Sales/Inventory controls facts, Billing controls charge, ERP Finance controls GL. Immutable correction chain: canonical `billable_occurrence_id` → `charge_id` → `reversal_of_charge_id` / delta adjustment → linked billing invoice credit note if closed. Imported events must not charge once locally and again after re-import under changed event ID. Same semantic ID with changed payload raises conflict.

### 20.5 AR-009 — Offline late-arrival and closed period, aligned with Mobile

Device `client_occurred_at` is **untrusted evidence**, not billing authority. Persist distinct `client_occurred_at`, `server_received_at`, `server_accepted_at`, `domain_effective_at`, `billing_received_at`, `rated_at`, `posted_at`, `usage_period_key`, `posting_period_key`, `pricing_version_id` and `original_charge_id?`. Mobile may queue locally but **never** debit SaaS wallet; backend reauthorizes current membership/device/permissions on replay and rejects revoked commands before billing occurrence. Server rejection = zero charge.

| Attribution option | Month selected | Strength | Risk |
|---|---|---|---|
| O: accepted domain effective | server-approved business transition month | economic alignment | historical close/re-rate adjustments |
| R: receipt | server first durable acceptance/receipt month | simple no backdating | penalizes delayed offline sync |
| C: verified client occurrence | validated timestamp if authoritative evidence exists | event economic timing | clock skew/forgery |
| **H: controlled hybrid (recommended PROPOSED)** | server-effective month while open; after close adjustment in current open posting month referencing original effective month | respects immutable close and event history | two-period reconciliation, tax/discount intricacy |

Illustration: device offline at **2026-10-31 23:58** local; sync receipt/acceptance **2026-11-01 09:00**, rated/posted Nov 1. Depending on authoritative server domain fact, economic October may be disputed; **do not adopt device October automatically**. H requires a verified effective timestamp, approved close cutoff and a separate immutable correction record. If October finalized, do not mutate it; post a November adjustment with `original_usage_period=2026-10`, `posting_period=2026-11`, `correction_reason` and invoice note. Month attribution, discount tier and negative grace effects remain OPEN until Business, Mobile, Sales and Finance sign off. Duplicate offline operations must result in at most one accepted economic occurrence.

### 20.6 MR-05 — Pricing/version precedence and resource tracking

Candidate evaluation flow (PROPOSED, precedence **OPEN**): verify source eligibility and BillingAccount mapping → select effective immutable price version (which effective timestamp? OPEN) → base document charge + over-threshold credits; or approved seat interval/resource measured unit → enterprise override at approved order → progressive tiers per approved window → deterministic fixed-point rounding → immutable rate breakdown & ledger. Credit/UZS conversion fixed at **APPROVED 1 credit = 10 so'm**.

| Pricing option | Benefit | Risk / approval |
|---|---|---|
| Enterprise overrides base then eligible discounts stack | flexible negotiated pricing | double-discount / stacking OPEN |
| Enterprise final net rate overrides standard discounts | predictable invoices | removes progressive benefit unless contracted OPEN |
| Prefer most-specific contract version for type/company | deterministic | conflict at overlapping intervals OPEN |
| Marginal tiers | smoother breakpoints | tier window and aggregation OPEN |
| Retroactive cumulative tiers | possible commercial simplicity | mid-month re-rating/reversal OPEN |
| Resource event-based unit billing | immediate metering | duplicate samples, API retries |
| Resource period snapshot (storage/time) | stable aggregate | cutoff/measurement version OPEN |

Resource Usage categories, inclusion/free thresholds, sample frequency, conversion, units and prices **OPEN**; cannot invent charges. Store measured quantity/unit, source proof, aggregation window, price version, contract version, discount policy version, intermediate amounts, rounding and charge period. Enterprise version or rate change never mutates posted invoice/ledger; corrections are explicit linked postings. No unapproved numeric rates or threshold.

### 20.7 Business decision packet — explicit approval required

| ID | Decision and options | Preferred PROPOSED | Main advantage | Main risk | User must approve |
|---|---|---|---|---|---|
| BD-BILL-R2-01 | A tenant pooled / B company account / C contract pooling / D branch | B | MVP audit simplicity | group payer complexity | LegalEntity→Company, debtor/payer and wallet cardinality |
| BD-BILL-R2-02 | N-current/N-next named month or A-anniversary clamp/rollforward | A-clamp as review candidate | closer full month after first negative | ambiguous business meaning | exact month rule, timezone, zero crossing |
| BD-BILL-R2-03 | full recovery resets vs holds negative window; repeat new vs original | retain historical window until policy-defined end | resists repeated reset | customer experience | top-up, expiry, repeat, post-expiry access |
| BD-BILL-R2-04 | membership writer Multi-Tenant vs Security; role writer Security | Multi-Tenant membership, Security grants | one authoritative writer | registry mismatch | AR-013 ownership, revocation contract |
| BD-BILL-R2-05 | seat invitation vs activation, suspend billing vs pause, company transfer, per-day/full-cycle | activated + day-proration candidate | intuitive charging | edge-case complexity | billable seat definitions, scope, dates and prices |
| BD-BILL-R2-06 | document transition per type; amendment/reversal/import charge | explicit versioned matrix | avoids duplicate documents | many cases | each chargeable type/status, unit, fee, threshold, refund/import |
| BD-BILL-R2-07 | O/R/C/H late attribution, adjustment in closed months | H candidate | immutable close | two periods | attribution/close/refund/discount/negative treatment |
| BD-BILL-R2-08 | marginal vs retroactive tier; enterprise override order | marginal, contract versioned | explanation/audit | contract stacking | tiers, windows, precedence, rounding |
| BD-BILL-R2-09 | resource measured events vs snapshots | metric-dependent | verifiable costs | overlaps | types, inclusion, units, collection cadence |

**No user approval implied.** Exact pricing, quantity breakpoints, device drift tolerance, timezone choice, service cut-off policy, revenue/tax treatment and periodic charges remain OPEN. Before implementation, sync approved choices into root registries by their owners; this Billing-only revision makes no global edits.

### 20.8 Round 2 acceptance gates

Test independent source and liability attribution; legal payer ≠ funding actor; stale mapping during company transfer; tenant/branch denial; same user multiple branches/devices counted once under approved scope; invited/suspended/revoked status; negative Oct 28 and Jan 31 boundary for both month models; verified top-up partial/full and repeat; closed October followed by delayed Nov event and correction; two offline devices same operation; external import idempotency; confirm→dispatch→return→amend without double base charge; enterprise tier overlap/rounding; resource time-window duplicate; membership revocation before sync; nonapproved service blocking cannot occur. **Acceptance cases are specification proposals, not tests performed.**

Related: [ADR-BILL-001](../adr/ADR-BILL-001.md), [ADR-BILL-006](../adr/ADR-BILL-006.md), [ADR-BILL-007](../adr/ADR-BILL-007.md), [ADR-BILL-009](../adr/ADR-BILL-009.md), [ADR-BILL-010](../adr/ADR-BILL-010.md).

## 21. Round 2 — foydalanuvchi tanlagan variantlar (2026-10-12; SELECTED_PENDING_APPROVAL)

**Governance:** foydalanuvchi beshta variantni “muhokama va rasmiy tasdiqni aniqlashtirish uchun” tanlagan. Bu **user-selected candidate**, ya'ni tasdiq emas: ushbu bo‘lim va ADRlar **PROPOSED**, barcha tegishli edge-case'lar **OPEN**. §20 dagi option tables saqlanadi; tanlangan variantni yangi APPROVED biznes qoidasi deb talqin qilmaslik kerak. [Audit Round 2](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md) va [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) yangicha APPROVED dalili bermaydi.

| Packet | Foydalanuvchi tanlovi | Recommendation & alternative | Main advantage | Main risk | Pending explicit approval |
|---|---|---|---|---|---|
| BD-BILL-R2-01 / AR-001, MR-01 | **Company 1:1 BillingAccount 1:1 Wallet**, candidate MVP | Tenant-pooled vs company-scoped vs enterprise pooled vs branch wallet | Payment liability and analytics simpler | Different LegalEntity, centralized payer and cross-company migration | Legally liable debtor, legal payer/source of funds, Company↔LegalEntity and account assignment |
| BD-BILL-R2-02 / MR-02 | **Anniversary-based one calendar month**, not named-month | Anniversary calendar arithmetic vs named current/next month | Similar grace relative to first negative | 29/30/31 dates, timezone, top-up/repeat | Clamp vs roll-forward, policy timezone, exact crossing, expiry and reset |
| BD-BILL-R2-05 / MR-03 | **Activated Seat with day proration** | Invite-billable vs activated; full cycle vs daily vs time fraction | Charges align with activation | Suspended/resumed seats, same user across company/branch | Effective date, day numerator/denominator, suspension and company transfers |
| BD-BILL-R2-07 / AR-009 | **Server-approved business-effective month; linked current-open-period adjustment if original period closed** | Receipt month vs client occurrence vs effective month/hybrid | Audit-friendly, immutable closed invoices | Negative grace and old-month discount reconciliation | Trusted effective event, cutoff, tier and grace impact, tax/invoice adjustment |
| BD-BILL-R2-08 / MR-05 | **Marginal progressive tiers** (each range its own discount) | Marginal vs retroactive entire-volume | No discontinuity at threshold | Contract stacking and aggregation period | Tier measurement window, enterprise priority, rates, rounding |

**Clarification of chosen month option (not new approval):** first negative 2026-10-28 16:00 → candidate expiry 2026-11-28 16:00; 2026-01-31 10:00 → Feb 28 10:00 **only if clamp approved**. Named-month alternative October ends 2026-11-01 00:00; next full named November month ends 2026-12-01 00:00. “One calendar month” must not be rewritten as 30 days. `first_negative_event_id`, effective billing timezone, `period_policy_version`, `period_start_at`, `period_end_at`, verified `topup_event_id?`, `recovered_at?` and retry lineage are proposed audit fields. Only **actual** billed Seat/Business Usage/Resource Usage entries may add negative debt; no estimated credit ceiling, quota or derived risk limit. Post-expiry read/write restrictions, repeat-negative and partially restored balance remain OPEN; do not silently suspend.

**MVP liability mapping:** separate `source_company_id/branch_id` from `billing_account_id` and `legal_debtor_entity_id/payer_party_id` as in §20.1. The payer who funds wallet need not be legal debtor. Cross-company transfer creates future-effective allocation snapshots; old immutable ledger stays linked to historical debtor; migration/transfer accounting requires Finance approval.

**Seat source and duplicate protection:** [Security §17](SECURITY_ARCHITECTURE.md) / [Multi-Tenant §19.4](MULTI_TENANT_ARCHITECTURE.md) propose **Identity→global credentials, Multi-Tenant→TenantMembership writer, Security→RoleAssignment/PermissionEvaluation, Billing→billable seat eligibility/charge**. This remains **AR-013 OPEN**, because the central MODULE_REGISTRY historically attributes membership to Security. Accepted/activated membership is only a fact, not automatic authorization to charge without approved Seat tariff. Stable membership ID + version + company effective allocation and interval dedup; company transfer and multi-branch membership cannot double-bill one contracted seat. Suspension and invitation charge options OPEN. No dual membership writer.

**Document matrix from §20.4 remains OPEN**: Sales Order, Sales Invoice, Purchase Order, Goods Receipt, Transfer, Return/Credit Note, Inventory Adjustment and imported documents each require separately approved state transition, base fee, excess-volume unit/threshold, reversal/refund and import replay behavior. Source Sales owns transition and canonical occurrence, Billing owns economic charge; do not confuse with stock or Finance GL.

**Late usage:** device clock is evidence only; Mobile does not charge offline and server checks live membership/device permission before accepting replay. The selected hybrid keeps `domain_effective_at`, `server_accepted_at`, `received_at`, `rated_at`, `posted_at`, `usage_period_key`, `posting_period_key`, `original_charge_id`; closed original months are never mutated. A current period adjustment references original effective month. Whether this changes original marginal tier, enterprise effective version or negative-month debt window stays **OPEN**.

**Resource / pricing:** no Resource Usage category, inclusion allowance, units, thresholds or numerical rate approved. Rate-order variants: enterprise override then standard discount vs enterprise net price; marginal tiers selected for review but stacking, ordering, overlapping contract, rounding, carry-forward and period window still OPEN. Persist immutable pricing/version/contract breakdown.

**Implementation gate:** selected preferences are neither ADR APPROVED nor production acceptance. Business + Multi-Tenant + Finance + Security + Sales + Mobile sign-off for their respective owner contracts and separate user confirmation required. No code authorized and no global registries changed here.

---

## 22. User-approved MVP Company liability va Tenant-shared Resource Usage (2026-10-12)

[ADR-BILL-011](../adr/ADR-BILL-011.md) — scoped authoritative decision record. **APPROVED_BY_USER:** MVP Company 1:1 distinct LegalEntity 1:1 BillingAccount 1:1 Wallet, Branch uchun alohida wallet yo'q; Company LegalEntity o'z SaaS qarzi uchun javobgar; authorized third-party Wallet top-up debt transfer emas; Seat user × Company bo'yicha, bitta Companyda Branch/device bo'yicha takrorlanmaydi; Tenant-shared haqiqiy Resource Usage oldindan tayinlangan bitta responsible Company hisobidan, ordinary Company/Branch usage o'z Company hisobidan. Eski §§3, 19–21 dagi aynan shu MVP cardinality/scope "not approved" jumlalari endi ushbu tor qism uchun tarixiy alternativ bo'lib qoladi; enterprise wallet pooling va boshqa opsiyalar APPROVED emas.

**PROPOSED / OPEN:** Tenant admin proposes, receiving Company finance representative accepts; versioned prospective-only assignment, no retroactive ledger mutation/no automatic Wallet transfer; next whole metering interval candidate only. No guessed billing account, unresolved source in PENDING_ATTRIBUTION. Third-party payer authorization evidence, refunds/disputes, resource classes/units/tariffs, billing time boundaries, Seat activation/day prorating, document charge eligibility, negative-month anchor and late offline effective month remain OPEN. Technical writer: Billing financial assignment/rating/Wallet; Multi-Tenant organization validation; Security auth; SaaS wallet remains separate from ERP Finance GL.

---

## 23. Seat lifecycle and daily proration contract (MR-03; 2026-10-12)

[ADR-BILL-012](../adr/ADR-BILL-012.md) defines the current **PROPOSED** (unapproved economic policy) state machine, company-day accrual identity, event/order/replay contract and acceptance gates. **Already APPROVED_BY_USER via [ADR-BILL-011](../adr/ADR-BILL-011.md):** one chargeable Seat identity per user × Company, no extra Seat per Branch/device, separate independently counted Seats for different Companies and distinct Company Wallet liability. Earlier §§19–21 scope alternatives about per-tenant/per-branch license are historical for MVP, not operative. Technical Membership writer and precise paid membership states remain separately signed off.

**Default PROPOSED:** INVITED/PENDING no Seat charge; ACTIVE membership can start daily-prorated Seat; effective-dated SUSPENDED/REACTIVATED/REMOVED generate recomputable intervals, but Security access denial never silently determines commercial license eligibility. Month price × approved billable local dates ÷ actual number of local dates in month, exact date-inclusion, timezone, charge cadence and rounding all require business approval. One canonical user×Company economic occurrence per day/segment; multiple membership IDs, roles, branches or devices never yield additional Seat. Company move snapshots old vs new legal debtor/Wallet. Source outbox + Billing dedup; same semantic business occurrence with different payload = conflict. Nothing in this section approves a specific Seat tariff, day-boundary policy, suspension billing, refund, month-negative policy or application code.

---

## 24. Delegated economic design selection — negative month + chargeable documents (2026-10-12)

[ADR-BILL-013](../adr/ADR-BILL-013.md) chooses `PROPOSED / DESIGN_SELECTED_UNDER_DELEGATION` anniversary-based one calendar month, end-of-month clamp, first actual posted negative crossing as start; no forecast or estimated credit ceiling. Local Company billing IANA timezone, persisted UTC boundaries, one contiguous episode grace, topup/recovery/rearm and post-expiry handling are **selected candidate**, not independently approved legal terms. Tested examples 31 Jan→28/29 Feb remain DESIGN, not runtime-verified.

[ADR-BILL-014](../adr/ADR-BILL-014.md) selects `PROPOSED` configured chargeability on SalesOrder.CONFIRMED, GoodsReceipt.POSTED, StockTransfer.COMPLETED (once per stable logical document); generated SalesInvoice, returns, historical imports and correction do not auto-add separate base; true measured over-threshold units and marginal progressive tiers with enterprise final-net override component scope. **CRITICAL:** no published type/transition/price/threshold/effective contract = **no Wallet debit**, despite candidate eligibility. Corrective reversal is immutable and approved by economic/legal policy, not automatic refund. Actual source fact, canonical occurrence and price version needed; device never rates or debits. Company Wallet/legal debtor and Tenant-shared assignment remain as explicitly approved in [ADR-BILL-011](../adr/ADR-BILL-011.md). [Handoff](../handoff/CODEX_START_HERE.md) tracks missing external tariff, fiscal and provider evidence; no coding authorization.
