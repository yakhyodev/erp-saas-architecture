# ADR-INT-003 — Read-first Staged Migration

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
Unverified source semantics and incomplete history make direct import risky.

## Variantlar
Direct upsert; raw DB copy; read-only staged import with reconciliation.

## Taklif etilgan yechim
Phased read-only discovery, staged validation, source traceability and signed cutover.

## Sabab
Auditability, reversibility and domain ownership.

## Xavflar
Storage and human approval workload.

## Boshqa modullarga ta'siri
Finance, Inventory, CRM, UX.

## Qaror sharti
Owning domain review required; no implementation authorization. See [main architecture](../modules/INTEGRATION_ARCHITECTURE.md).
