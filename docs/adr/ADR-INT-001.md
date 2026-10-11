# ADR-INT-001 — Anti-Corruption Adapter Layer

- Date: 2026-10-11
- Status: **PROPOSED** (NOT APPROVED)
- Owner: Integration Architecture

## Muammo
External SalesDoc request/ID semantics might leak into ERP core.

## Variantlar
Direct calls from each module; full source-schema mirror; shared adapter/ACL.

## Taklif etilgan yechim
Shared provider-neutral adapter plus canonical owner-domain commands.

## Sabab
Separation, consistency and future provider extensibility.

## Xavflar
Mapping cost, potential central orchestration bottleneck.

## Boshqa modullarga ta'siri
Backend, CRM, Sales, Inventory, Finance, Security.

## Qaror sharti
Owning domain review required; no implementation authorization. See [main architecture](../modules/INTEGRATION_ARCHITECTURE.md).
