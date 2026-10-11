# FINANCE_ACCOUNTING.md
Status: PROPOSED. Date: 2026-10-11.
Purpose: Detailed accounting boundaries and finance controls for the multi-tenant ERP SaaS.

## Scope
Double-entry General Ledger, chart of accounts, cashboxes, banks, AR, AP, payment allocations, financial invoices, journal audit, reconciliations, reversals, multi-currency and branch-level reporting.

## Approved requirements
- ERP customer Finance Ledger and SaaS Platform Billing Ledger are distinct accounting domains.
- Multi-company and branch management operate within the SaaS platform.
- SaaS monetization is Hybrid Prepaid: low-cost Seat + Business Usage + Resource Usage; 1 credit = 10 UZS; progressive discounts and enterprise contracts; actual charges may take wallet negative for one calendar month without estimated credit ceiling. These apply to **platform billing**, not customer Finance accounting.
- Interface languages: Uzbek, English, Russian, plus hidden future Uzbek alphabet option.

## Proposed design
1. Accounting Book per legal entity, with explicit branch dimensions; legal entity model needs approval.
2. Double-entry journals with debit/credit equality, immutable posting and auditable reversals.
3. AR/AP open-item subledgers reconciled to GL control accounts.
4. Cash and bank transactions supported by source documents and settlement references.
5. Invoice and payment workflows with partial allocations, advances, refunds and credit notes.
6. Multi-currency recording of transaction amount, functional equivalent, rate source and FX differences.
7. Close periods with maker-checker controls and explicit reopening.
8. Tenant isolation, least privilege, audit metadata and reconciliation exception queues.
9. Financial reports by company/legal entity, branch and consolidated group only when authorized.

## Key inter-module contracts
Sales and Purchase produce validated business facts; Finance alone owns posting rules and accounting entries. Inventory owns quantities and valuation events; Finance owns GL impact. Security validates actor and scope. Billing cannot mutate ERP journals.

## Risks
Tax/regulatory rules not yet selected; double posting from asynchronous delivery; FX rounding; reconciliation drift; multi-branch legal entity ambiguity; access control leakage; correction workflows.

## Open questions
Jurisdiction, legal entity hierarchy, functional currency, invoice posting point, inventory valuation approach, bank import formats, payment settlement authority, period close policy.

## ADR references
FIN-001: separate Finance/Billing ledgers (approved requirement). FIN-002: immutable double-entry ledger (proposed). Do not interpret other proposed decisions as approved.

## Cross-module contract reconciliation (AR-004, AR-005, AR-008, MR-06) — 2026-10-11

**All new rules PROPOSED; open policy decisions remain OPEN.** This section supplements, not supersedes, the earlier design.

### Distinct accounting obligations

| Field | Tenant ERP Finance | SaaS Platform Billing |
|---|---|---|
| Obligation | Customer invoice debtor / supplier creditor | Contracted BillingAccount payer, mapping AR-001 OPEN |
| Source | Sales, purchase, inventory and settled payments | Seat, Business Usage, Resource Usage and top-ups |
| Ledger | Tenant legal-entity GL, AR/AP, cash/bank | Independent SaaS wallet/charge ledger |
| Monetary semantics | Currency, tax, fiscal periods under pending accounting policy | 1 credit = 10 UZS; actual charges may go negative for one calendar month; no estimated limit |
| Mutation authority | Finance posting engine only | Billing posting engine only |
| Cross-boundary exchange | Explicit verified references or independently authorized financial documents | Usage facts are not ERP GL entries |

**FIN-001:** Existing ADR remains `APPROVED` as written. `DECISIONS.md` and architecture review flag missing independent explicit approval provenance (AR-004 OPEN). Do not treat this addendum as new approval.

### Proposed source-to-accounting contracts

| Event | Source owner | Finance obligation | Required dedup reference |
|---|---|---|---|
| SalesOrderConfirmed | Sales | No automatic AR/GL; optional future credit exposure validation | order transition ID |
| StockDispatchPosted | Inventory | Candidate cost/stock GL posting; timing and valuation OPEN | stock movement ID + journal kind + policy version |
| SalesInvoicePosted | Finance | Create debtor AR open item and balanced journal if policy permits | invoice posting operation ID |
| VerifiedPaymentSettled | Payment integration/Finance | Create cash/bank entry and authorized allocation | verified settlement ID |
| ReturnStockPosted | Inventory | Candidate valuation reversal; never implicitly reverse invoice | return movement ID + original reference |
| CreditNotePosted | Finance | Adjust AR and GL via linked compensating journal | credit note ID + original invoice |
| PartyMerged | CRM/shared Party candidate | Preserve historical legal debtor snapshot and alias lineage | canonical party merge/version |
| BillableOccurrenceCandidate | Sales / source domain | No ERP GL effect | canonical occurrence ID; Billing owns rating |

### Identity and settlement rules (PROPOSED)

`Party` is canonical identity candidate; `Customer` is CRM relationship, `Outlet` delivery/commercial location, `Payer` remitter, and `ReceivableOwner` the legally liable debtor captured on a Finance invoice. They must not be assumed equal. Finance owns AR open-item accounting, not the global Party master. Payment by a third party requires explicit authorization and traceable allocation; party merges never rewrite posted legal snapshots. AR-008 canonical owner and merge rules remain OPEN.

### Finance lifecycle safeguards (PROPOSED)

- Order draft/confirm: no physical stock decrement or GL/AR posting by default; reservation is not physical dispatch.
- Dispatch: Inventory posts stock once; Finance recognizes cost/stock only after approved accounting trigger; no invoice created by implication.
- Invoice: Finance posts AR/revenue/tax only under chosen tax and recognition policy; invoice never changes stock.
- Settled payment: payment allocation reduces AR exactly once; partial/advance/unapplied cases retain independent balances.
- Return: Inventory accepts physical stock; Finance separately issues credit note/refund/reversal if authorized; never automatic SaaS refund.
- Reversal: compensating immutable entries reference originals; retry produces same result; cross-tenant or different-payload replay is rejected.
- Reconciliation: compare invoice AR open items to GL control, stock valuation to GL, bank statement to cashbook, and Finance references to source facts; report unmatched/late items, never silently manufacture entries.

**Required posting envelope (PROPOSED):** tenant_id, company_id, legal_entity_id?, branch_id?, source_domain, source_document_id, source_fact_id, event_id, aggregate_version, operation_id, journal_kind, accounting_policy_version, occurred_at, committed_at, correlation_id, causation_id?, currency, amount, payload_digest, original_fact_id?. Durable inbox and unique economic posting key prevent duplicates; authorization and effective legal-entity mapping must be verified server-side. Finance journal and dedup marker are one local atomic transaction; cross-domain transaction model OPEN.

### Outstanding consent gates

- **AR-004:** FIN-001 approval evidence/source.
- **AR-005 / MR-06:** dispatch vs invoice vs delivery accounting trigger, cost method, partial shipments and credit-note policy.
- **AR-008:** Party/Customer/Payer/ReceivableOwner owner and identity/merge policy.
- **FIN-OPEN-TAX:** country jurisdiction, tax recognition, invoice/legal numbering.
- **FIN-OPEN-FX:** legal-entity functional currency, FX source, revaluation and rounding.
- **FIN-OPEN-PERIOD:** fiscal period calendar, close/reopen and late posting.
- **FIN-OPEN-ENTITY:** legal entity versus branch accounting books.
- **FIN-OPEN-ATOMICITY:** Finance posting synchronization and failure recovery.

**References:** [Finance Architecture](FINANCE_ARCHITECTURE.md), [Sales & Inventory](SALES_INVENTORY.md), [CRM](CRM_DISTRIBUTION.md), [Billing](BILLING_ARCHITECTURE.md), [FIN-001](../adr/FIN-001.md), [FIN-002](../adr/FIN-002.md), and root `DECISIONS.md`/`OPEN_QUESTIONS.md`. No shared file is modified here.
