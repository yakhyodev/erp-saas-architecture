# ADR-INT-003 — Read-first Staged Migration

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
External source semantics, history completeness and deletion behavior are not fully verified.

## Proposed migration contract
Stages:
DISCOVERED → STAGED → VALIDATED → MAPPED → ACCEPTED → POSTED → RECONCILED → ARCHIVED.

Each stage records source provenance, contract version, evidence level, checkpoints and reconciliation results.

## Rollback evidence
Cutover requires snapshot/checkpoint, reconciliation evidence and owner acceptance. Rollback must prevent duplicate documents.

## Variantlar
Direct import; raw copy; staged migration.

## PROPOSED yechim
Read-first staged migration.

## Status
No live migration performed or verified.
