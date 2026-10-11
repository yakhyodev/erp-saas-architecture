> **Historical PROPOSED concept.** ADR-UX-001..003 labels in this document are preliminary and are not canonical. The authoritative ADR-UX-001..006 are defined in `UX_ARCHITECTURE.md` and separately under `docs/adr/`.

# UX/UI Architecture

Status: PROPOSED
Date: 2026-10-11
Scope: ERP SaaS user experience, navigation, interface standards and performance budgets.

## Goals and boundaries
- Simplify discovery of ERP features through task-oriented navigation.
- Define role-aware dashboards, global search, quick actions, data tables, keyboard workflows, accessibility and responsive patterns.
- UX never replaces backend authorization, billing logic, business workflows or tenant isolation.
- Company/branch context must be visible and consistent; actual access is enforced by backend.

## Proposed information architecture
- Persistent company/branch context switcher, global search, notifications and user settings.
- Task-oriented primary navigation: Dashboard, Sales, Inventory, Finance, CRM, Reports; modules shown only when available and permitted.
- Contextual secondary navigation within each module, breadcrumbs on deeper screens.
- Progressive disclosure for advanced settings, without hiding essential actions.
- A single prominent primary action per task-focused screen where appropriate.

## Proposed dashboards
- Role/persona examples (not approved RBAC roles): owner, sales manager, warehouse operator, accountant, branch manager.
- Each dashboard: priority tasks, actionable exceptions, relevant KPIs, recent documents.
- KPI access and drill-down respect tenant, company, branch and permission scope.
- Widget customization is optional and not approved for MVP.

## Global search and quick actions
- Search across authorized documents, clients, products and navigation actions.
- Search results grouped by type, with keyboard selection and clear no-results states.
- Never expose unauthorized names, counts, previews or cross-tenant suggestions.
- Quick actions available from a command palette and visible page controls.
- Suggested shortcut: Ctrl/Cmd+K for command palette, subject to conflict testing.

## Large tables
- Server-side filtering, sorting, pagination or cursor navigation depending on domain and consistency needs.
- Explicit filter chips, saved views (proposed), column visibility, bulk selection with permission checks.
- Virtualization considered for large visible result sets; avoid loading entire tenant datasets into the browser.
- Stable row identity, loading/empty/error states, keyboard navigation, export permissions and clear bulk-action confirmation.

## Accessibility and localization
- Target WCAG 2.2 AA; keyboard reachability, focus visibility, semantic labels, adequate contrast, screen-reader announcements.
- Responsive layouts: desktop tables, tablet compact tables, mobile task-first cards or horizontal table view where required.
- Approved locales: Uzbek, English, Russian; hidden future Uzbek alphabet option must not appear as a launched locale.
- Localized date, number, currency and text expansion; do not encode workflow state only in color.

## UX performance budgets (PROPOSED, not SLA)
- User action visual acknowledgement: ideally <= 100 ms.
- Common in-app navigation: useful content ideally <= 1 s on a typical connection and warm session.
- Filter/search interactions: visible feedback immediately; results ideally <= 1 s for common queries.
- Initial critical screen: target LCP <= 2.5 s at p75 where applicable; INP <= 200 ms, CLS <= 0.1, measured with realistic devices and networks.
- Longer operations show progress and allow safe retry where supported; budgets require validation with backend teams.

## User flows / wireframes
1. Login -> choose authorized company/branch -> role-aware dashboard -> task -> document -> confirmation -> updated state.
2. Global search -> query -> grouped authorized results -> detail -> context-preserving back navigation.
3. Sales list -> filter -> select document -> action -> confirmation -> status feedback.
4. Mobile -> company context -> task shortcut -> focused form -> validation -> confirmation.

Desktop shell: sidebar | top bar (company/branch, search, notifications, account) | content (title, primary action, filters, table/cards).
Mobile shell: compact header, task-based navigation, search and quick actions, responsive content.

## ADR-UX-001 — Task-oriented shell (PROPOSED)
Problem: Feature-heavy ERP navigation makes common actions difficult to find.
Options: deep module menus; task-oriented shell; fully customizable navigation.
Decision proposed: task-oriented shell with contextual secondary navigation and search.
Reason: discoverability and consistency without premature customization complexity.
Risks: some advanced tasks require extra navigation; role taxonomy needs alignment.
Impact: RBAC, module registry, routing, localization, analytics.

## ADR-UX-002 — Authorization-aware search (PROPOSED)
Problem: Cross-module search must be fast without exposing tenant data.
Options: client-side aggregation; dedicated search service; backend-orchestrated search.
Decision proposed: backend-authorized federated search initially, with later indexing if measured need arises.
Reason: minimize early operational complexity and centralize authorization.
Risks: latency across modules, inconsistent ranking, eventual consistency if indexing is later introduced.
Impact: backend, database, tenant isolation, CRM, sales and inventory.

## ADR-UX-003 — Large-data table patterns (PROPOSED)
Problem: High-volume document lists can be slow and difficult to operate.
Options: load all rows; server-driven pagination/filtering; universal virtualization.
Decision proposed: server-driven queries, explicit filters, optional virtualization where useful.
Reason: bounded resource usage and predictable interactions.
Risks: costly counts, complex bulk selections, export limits.
Impact: API contracts, indexing, audit logging and permissions.

## Open questions
- Which roles and permissions will be approved in the central RBAC model?
- Which modules and actions belong in MVP navigation?
- Are saved table views per user, per role, or per company?
- How should cross-branch aggregate views be authorized and presented?
- Which devices and connection profiles define performance acceptance?
- Are offline mobile flows required, and for which operations?
- Which global shortcuts can be reserved without browser/OS conflicts?

## Synchronization plan (not yet applied)
- DECISIONS.md: register ADR-UX-001..003 only upon approval, with final status.
- MASTER_ARCHITECTURE.md: reference navigation shell, authorization-aware search and performance approach after approval.
- REQUIREMENTS.md: record agreed accessibility, localization, responsive and performance acceptance criteria.
- MODULE_REGISTRY.md: link UX to RBAC, search, CRM, sales, inventory, finance, reporting and billing.
- OPEN_QUESTIONS.md: track unresolved UX cross-module decisions.
- CHANGELOG.md: add dated entries when decisions are approved or revised.