# Membiqo — Copy/Paste AI Development Prompts

Use these prompts with a coding agent (Cursor, Claude Code, Copilot coding agent, etc.) **after putting all markdown documents in the repository root**. Prefer a new prompt per milestone, not one request to create everything.

## Prompt 1 — Bootstrap (start here)

```text
You are my senior full-stack engineer. We are building Membiqo, a multi-tenant membership-management SaaS for an exhibition.

Read AGENTS.md, decisions.md, prd.md, architecture.md, database.md, api.md, security.md, ux.md, roadmap.md, testing.md, and deployment.md fully before coding. Do not assume an existing implementation.

Implement ONLY Milestone 0 from roadmap.md:
- Create Next.js App Router + strict TypeScript + Tailwind CSS + shadcn/ui foundation.
- Add Supabase browser/server client utilities, environment validation, lint/typecheck/test scripts, and a useful responsive shell.
- Create a polished Membiqo landing and pricing page advertising ₹0/month beta. Do not add any real payment gateway or checkout.
- Set up unit and E2E test infrastructure, README instructions and .gitignore.

Use maintained compatible dependency versions. Give me the planned file tree before major changes, implement incrementally, run tests, then report files changed, commands/results, security decisions and next issue. Do not invent completed tests.
```

## Prompt 2 — Auth and tenants

```text
Read AGENTS.md and all relevant documents. Implement ONLY Milestone 1: Supabase authentication, profiles, business onboarding, owner role, free_beta subscription, tenant switcher, database migrations and RLS. Business creation must be atomic. Add negative tests proving owner A cannot see owner B's tenant. No payment UI. Report test results and known limitations.
```

## Prompt 3 — Portals, members and plans

```text
Read the project specs. Implement ONLY Milestone 2. Use one codepath for gym, yoga, martial_arts, coaching_center, study_center, sports_academy; vary labels from a preset config. Build public business portal, owner plan CRUD, member registration with pending approval, manual owner-created members, and role-scoped views. Do not collect Aadhaar or medical data. Add RLS and API tests for every CRUD route.
```

## Prompt 4 — Membership orders

```text
Implement ONLY Milestone 3 based on the docs: support zero-cost member plans; optional business-owned UPI QR/cash with owner manual confirmation; no integrated gateway. Implement order price snapshot, registration approval precondition, transactional idempotent activation, exclusive membership end timestamps, calendar month/year clamping, and overlap checks. Add unit/integration tests for concurrent approval and expiry edge cases. Give precise verification commands.
```

## Prompt 5 — Cards and secure check-in

```text
Implement ONLY Milestone 4: generate authorized PDF ID cards; secure opaque verification tokens; public verifier with no PII; rotating 60-second server-created QR attendance challenges; signed-in member camera scan; active membership check and one check-in per business-local day. Ensure tenant isolation. Add tests for expired/replayed/other-tenant QR and leaked private data.
```

## Prompt 6 — Dashboard, admin and demo

```text
Implement ONLY Milestones 5 and 6 in separate commits: tenant-scoped KPIs, reports, restricted admin overview, audited tenant suspension, polished mobile UX, synthetic exhibition seed, deployment guide, responsive and E2E tests. No gateway. Finish by walking through the exact demo script in ux.md and reporting pass/fail of every critical test in testing.md. If a blocker exists, explain it without claiming success.
```

## Debugging prompt

```text
Fix this issue in the Membiqo codebase: [PASTE ERROR OR REPRO]. First identify the root cause and affected tenant/role paths. Read AGENTS.md, decisions.md and relevant docs. Propose a minimal fix. Do not disable RLS or bypass authentication. Add a regression test and show commands/results.
```

## UI improvement prompt

```text
Improve Membiqo's current screen [ROUTE] using ux.md. Keep existing data semantics, permission checks and accessibility. Use responsive layouts, consistent tokens, clear empty/loading/error states. Avoid changing APIs or business rules without tests. Explain visual changes and verify mobile behavior.
```

## Security audit prompt

```text
Act as a security reviewer for Membiqo. Review all RLS policies, server route handlers, tenant joins, composite FKs, service-role usage, card verification, private uploads, QR check-in, and admin endpoints against security.md and testing.md. Report concrete vulnerabilities with severity, reproduction steps using synthetic users, proposed fixes and missing tests. Do not claim the application is secure without evidence.
```
