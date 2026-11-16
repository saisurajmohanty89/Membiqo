# Membiqo — AI-Friendly Implementation Roadmap

**Instruction:** Complete milestones in order. Each milestone must include migration/code, tests, a local verification checklist, and an end-of-task summary. Do not generate the entire app in one enormous AI prompt.

## Milestone 0 — Repository foundation

**Tasks:** Initialize Next.js + TypeScript + Tailwind + shadcn/ui; configure lint, formatting, env validation, Vitest and Playwright; create reusable layout/brand tokens; create Supabase browser/server clients with no secret leakage.

**Deliverable:** Landing page shell, auth forms shell, navigation, env example, working typecheck/lint/test scripts.

**Acceptance:** `npm run lint`, `npm run typecheck`, `npm run test` and dev server work; `.env.local` is ignored; marketing shows **₹0 free beta**; no real checkout.

## Milestone 1 — Auth and tenant onboarding

**Tasks:** Database migrations for profiles, businesses, business_users, business_subscriptions and platform_admins; secure auth; business creation as atomic operation; slug validation; tenant switcher; role guards and initial RLS; basic account UX.

**Acceptance:** Two owners create distinct businesses and each sees only their own; `free_beta` subscription exists at 0 paise; duplicate slug cleanly rejected; negative RLS tests pass.

## Milestone 2 — Portal, members, and plans

**Tasks:** Public `/b/[slug]` portal, preset-specific labels, plan CRUD, member self-registration, owner manual member creation, approve/reject member, private photos, responsive lists and forms.

**Acceptance:** Gym and coaching portals share implementation; a member's registration starts pending; user A cannot read B's tenant member data; no Aadhaar or medical fields.

## Milestone 3 — Orders and activation

**Tasks:** `membership_orders` and `memberships` migrations; zero-price order path; optional owner-provided UPI QR and proof; cash confirmation; manual approval/reject; atomic idempotent activation; plan duration arithmetic; member status screen.

**Acceptance:** Free orders need no payment screenshot; no false payment success; approved order creates one membership and repeat approval does not create two; exact expiry tests pass.

## Milestone 4 — Cards and QR attendance

**Tasks:** Secure PDF card endpoint; opaque verification token and minimal public page; staff rotating short-lived attendance QR; member scanner; server-side eligibility and duplicate check-in; audit logs.

**Acceptance:** Member can download and verify card; fresh QR checks in active member once per day; expired QR, expired membership, other business, and duplicate attempts are denied; public verifier reveals no PII.

## Milestone 5 — Dashboards, platform admin, polish

**Tasks:** Business KPIs/reports, pending-order counters, expiry lists, admin tenant list, suspend/reactivate with audit, empty states, accessibility, seed data, docs and deployment.

**Acceptance:** End-to-end exhibition script in `ux.md` runs on a phone and desktop; all P0 cases in `testing.md` pass; no active subscription gateway.

## Milestone 6 — Exhibition deployment and rehearsal

**Tasks:** Staging/production envs, database backups, HTTPS, test accounts and synthetic data, release checklist, QR projector tests, error monitoring.

**Acceptance:** Every P0 exhibition journey works on target devices, tenant isolation is independently tested, and a rollback plan exists.

---

## Suggested implementation issues (ordered)

| ID | Title | Depends on |
|---|---|---|
| M0-01 | Repo + tooling + CI | — |
| M0-02 | Brand shell + marketing/pricing | M0-01 |
| M1-01 | Auth and profile migration | M0-01 |
| M1-02 | Businesses / roles / free plan migration | M1-01 |
| M1-03 | Business onboarding and tenant RBAC tests | M1-02 |
| M2-01 | Preset config + public portal | M1-03 |
| M2-02 | Member registration + manual entry + RLS | M2-01 |
| M2-03 | Plans CRUD + validation | M2-01 |
| M3-01 | Order schema + free/manual request flow | M2-02, M2-03 |
| M3-02 | Activation transaction + date rules | M3-01 |
| M4-01 | Card generation + verifier | M3-02 |
| M4-02 | Rotating challenge + member scan | M3-02 |
| M5-01 | Owner KPIs | M3-02, M4-02 |
| M5-02 | Admin panel + audit | M1-03 |
| M5-03 | QA + accessibility + documentation | All prior |
| M6-01 | Deploy + demo rehearsal | M5-03 |

## AI task completion format

For each issue, ask the agent to return:

1. **Files changed** and migration IDs.
2. **Security boundary** and how tenant isolation was enforced.
3. **Commands run** and results (or why not run).
4. **Manual test steps**, with both success and unauthorized cases.
5. **Remaining limitations**, TODOs and next issue.

## Hard stop rules

Pause any implementation that requires introducing a real payment gateway, billing real businesses, collecting Aadhaar, accessing unassigned tenant data, or weakening RLS; these require founder review and PRD updates.
