# ADR-INT-004 — Single Writer and Guarded Outbound Sync

- Date: 2026-10-11
- Status: **PROPOSED**
- Owner: Integration Architecture

## Muammo
Uncontrolled bidirectional writes can duplicate orders, stock and financial events.

## Proposed contract
Each synchronized process defines:
- authoritative writer;
- allowed direction;
- idempotency key;
- reconciliation evidence;
- rollback procedure.

Outbound writes remain disabled until explicitly approved by owning domains.

## Variantlar
Two-way sync; read-only forever; guarded single writer.

## PROPOSED yechim
Process-scoped single writer with controlled exports.

## Status
NOT APPROVED. No production sync enabled.
