# ADR-INT-004 — Single Writer and Guarded Outbound Sync

- Date: 2026-10-11
- Status: **PROPOSED**
- Owner: Integration Architecture

## Muammo
Blind bidirectional writes may double-post orders, stock or payments.

## Ko'rib chiqilgan variantlar
Always two-way; perpetual read-only; process-scoped single writer with opt-in exports.

## Taklif etilgan yechim
Choose one source of truth per process, outbound disabled until explicitly approved.

## Sabab
Avoid conflicts, duplicate postings and ambiguous timeout replay.

## Xavflar
Constraints on transition coexistence.

## Boshqa modullarga ta'siri
Sales, Finance, Inventory, Security, Billing.

## Tasdiqlash sharti
Domain owners review; this is NOT APPROVED. See [Integration Architecture](../modules/INTEGRATION_ARCHITECTURE.md).
