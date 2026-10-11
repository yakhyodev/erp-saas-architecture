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
