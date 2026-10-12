# ADR-UX-006 — Responsive parity va a11y

**Status:** PROPOSED
**Source:** [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md)

- **Status:** PROPOSED
- **Muammo:** mobil va desktop turli, topilishi qiyin ish usuliga aylanadi.
- **Variantlar:** desktop-first shrink; alohida navigatsiya logikasi; bir xil IA/business flows, device-specific presentation.
- **Tanlov:** shared task taxonomy, device-specific layout, WCAG 2.2 AA audit target.
- **Sabab:** xodimning qurilma almashtirganda qayta o'rganmasligi.
- **Xavf:** mobile murakkab jadval, offline/assistive tech edge case.
- **Ta'sir:** Mobile/Offline, Localization, Frontend, QA.

## MR-20 responsive/a11y evidence — PROPOSED

Same business task taxonomy on desktop/mobile, adaptive layout and explicitly marked offline availability. Distinguish draft, offline queued, accepted, rejected, conflict and revoked credentials. Proposed WCAG 2.2 AA manual+automated checks: keyboard/focus, screen-reader error/status, zoom/reflow, contrast, accessible tables/dialogs and touch targets. Public locales uz/en/ru; future alphabet remains hidden internal. p75 LCP <=2.5s, INP <=200ms, CLS <=0.1 and discovery <=3 interactions are targets only, never measured success. Device/network/sample and acceptable completion thresholds OPEN MR-20. See [UX §16](../modules/UX_ARCHITECTURE.md), [Mobile](../modules/MOBILE_ARCHITECTURE.md), [Localization](../modules/LOCALIZATION_ARCHITECTURE.md).

## Round 2 UX correction — AR-012 / MR-16/17/20 (PROPOSED)

Round 2 MR-20: Desktop/mobile share task taxonomy, differ in layout. Three public languages uz/en/ru; hidden future alphabet stays OFF. WCAG 2.2 AA proposed with keyboard/screen-reader/zoom/reflow/touch, manual+automatic checks. Offline queued != accepted; revoke denies sync. Web Vitals LCP≤2.5s INP≤200ms CLS≤0.1 p75 and discovery≤3 are PROPOSED budgets, not achieved metrics. Device/network pilot profile OPEN.

Details: [UX architecture §17](../modules/UX_ARCHITECTURE.md). No tests performed; no approved new technical decision.
