# ADR-CRM-004 — Historical staff and territory assignments

- **Status:** PROPOSED
- **Date:** 2026-10-11

## Problem
Reassigning an agent, supervisor or outlet must not rewrite history or invalidate KPI reporting.

## Alternatives
Overwrite direct owner fields; effective-dated versioned relationships.

## Proposed decision
Model agent–supervisor, agent–territory, outlet–territory and branch coverage using effective-from/to assignments with audit and conflict validation. Attribute visits and KPIs using relations in force when the event happened.

## Reason
Preserves historical traceability and multi-branch flexibility.

## Risks
Overlapping assignments and more complex temporal queries.

## Other modules
Database, Identity, Sales, Performance, Analytics.

**Not approved; no code authorized.**

## Round 2 alignment — MR-09/MR-13 (PROPOSED; 2026-10-12)

Agent, supervisor, outlet, company and territory relationship records require effective-from/to, tenant/company authorization and immutable audit. Customer/Outlet/Party role references are distinct; agent assignment never grants global Party access or changes legal debtor. UX-T08/T09 searches and route/visit selections must recheck current scoped grants; stale offline assignment is a conflict requiring reconciliation, not silent overwrite. **OPEN:** overlapping territories, outlet-to-customer cardinality and historical reassignment approver. This is not a new approval.
