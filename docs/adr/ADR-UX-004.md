# ADR-UX-004 — Workflow-first business actions

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** foydalanuvchi hujjat turini tushunmaguncha amaliyotni boshlay olmaydi.
- **Variantlar:** generic CRUD form; wizard; kontekstga mos guided flow.
- **Tanlov:** yuqori chastotali task uchun guided flow, power-user alternativasi.
- **Sabab:** onboarding va xatoni kamaytirish.
- **Xavf:** murakkab biznes senariylarida qadam ko'payishi.
- **Ta'sir:** Sales/Inventory, Finance, CRM state machine va validators.

## AR-012 MR-16 top-20 workflows — PROPOSED

The [persona × 20 candidate tasks](../modules/UX_ARCHITECTURE.md) are not approved MVP scope. Per task record member/scope, device, locale, network, expected business outcome, prohibited transition, error recovery, task-discovery steps, time, success and domain approval. <=3 actions measures finding/initiating a task, not finishing a multi-step workflow. Queued/draft does not mean posted, paid, stock-decremented or SaaS-chargeable. Gate G0/G2/G3/G5 remains OPEN; Sales/Finance/Billing/Mobile own actual transitions and AR-005/009/010 are unresolved. Status PROPOSED.
