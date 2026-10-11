# FINANCE_ARCHITECTURE.md
Status: PROPOSED (except explicitly approved business requirements)
Scope: ERP Finance & Accounting. Date: 2026-10-11.

## Domain boundaries
ERP Finance accounts for each customer's sales, purchases, cash, bank, AR, AP and financial statements. SaaS Platform Billing independently accounts for seat subscriptions, business usage, resource usage, wallet credits and SaaS charges. **APPROVED requirement:** these ledgers must remain separate. No cross-ledger direct posting.

## Proposed accounting architecture
- Double-entry journal engine: every posted journal balances debit and credit in functional currency; immutable posted entries.
- Chart of accounts configurable per legal entity; accounting periods and controlled closing.
- Source-document references, idempotent posting, transactionally consistent journal header and lines.
- Cash and bank: cashboxes, bank accounts, receipts, disbursements, transfers, bank statement import and matching.
- Receivables: customer open items, aging, partial settlement, advance payments, credit notes, write-offs with approval.
- Payables: supplier open items, due dates, partial settlements, advances and approval-controlled payments.
- Payment lifecycle: draft, authorized, submitted, settled, failed/reversed; allocate only settled funds according to policy.
- Invoice lifecycle: draft, issued, posted, partially paid, paid, cancelled via reversal/credit note. Separate business invoice from SaaS invoice.
- Multi-currency: transaction currency, functional currency, rate source/time, realized/unrealized FX gains and losses.
- Branch dimension: branch-coded entries with legal entity boundary; interbranch settlement requires explicit policy.
- Audit: actor, tenant, legal entity, timestamps, source and reason; no deletion of posted journal records.
- Reconciliation: bank-to-cash, AR/AP-to-GL, inventory-to-GL, suspense accounts and period-end discrepancies.
- Financial reports: trial balance, balance sheet, P&L, cash flow, AR/AP aging, branch reporting.

## Integrations
Sales/Purchase/Inventory own operational documents; Finance owns accounting policy and postings. Tenant/Security owns identity and permissions; Database owns isolation and integrity; Billing owns SaaS wallet and usage ledger. Domain events must be idempotent and replay-safe.

## Invariants (PROPOSED)
1. No cross-tenant postings.
2. Debits equal credits for each posted journal.
3. Posted journals are immutable; reversal and corrective entries retain linkage.
4. Closed periods reject postings unless reopened by authorized workflow.
5. Original currency, conversion rate and functional-currency amounts remain auditable.
6. Every payment allocation refers to a valid open item; allocations cannot exceed permitted outstanding balances.
7. Finance and platform billing ledgers never share balances or entries.

## Open questions
Legal entity versus branch accounting boundaries; Uzbekistan-first or international tax jurisdictions; functional currency; posting triggers; FX rate policy; close/reopen authority; interbranch clearing; bank integration scope.

## Codex architecture specification (no code)
Define modules for accounting policy, journal posting, cash/bank, receivables, payables, payment orchestration, invoices, FX, reconciliation and audit. Specify entities, state machines, domain events, transactional boundaries, tenant-scoped permissions, idempotency, reversal workflows, financial reporting read models and invariant tests before implementation. Do not hardcode unresolved accounting or tax policies.

## ADR links
See docs/adr/FIN-001.md and docs/adr/FIN-002.md. All implementation choices remain PROPOSED pending approval.
