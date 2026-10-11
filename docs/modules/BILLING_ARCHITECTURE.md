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
