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
