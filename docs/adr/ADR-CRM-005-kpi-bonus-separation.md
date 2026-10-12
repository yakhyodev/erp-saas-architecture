# ADR-CRM-005 — KPI results and bonus payments

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** CRM & Distribution
- **Related module:** [CRM_DISTRIBUTION.md](../modules/CRM_DISTRIBUTION.md)

## Problem
Metric recalculations could unintentionally create financial obligations.

## Options considered
KPI engine posts directly; versioned results and approval handoff.

## Proposed decision
KPI engine produces versioned results and bonus proposals, with approval; HR/Finance alone posts and pays.

## Rationale
Separation of duties and traceable corrections.

## Risks
Late data, disputes, mismatched reporting and payout timing.

## Impact on other modules
Finance, HR, Sales, Security, Analytics.

## Approval
Not approved. Requires cross-module architecture review. No application code authorized.

## Round 2 alignment — MR-13 (PROPOSED; 2026-10-12)

KPI attribution uses effective-dated agent/outlet relations, verified visit evidence with explicit missing/low-accuracy statuses and authoritative Sales/Finance outcomes. Party merges must not rewrite historical KPI inputs or posted Finance debtor snapshots. Pending offline evidence is provisional; photo/GPS retention policy governs evidence separately from metric totals. Bonus approvals and actual payments belong to HR/Finance, not CRM. **OPEN:** consent/retention legal basis, metric formula/period cutoff, late-sync and reversal handling, approvers. No code or test execution claimed.
