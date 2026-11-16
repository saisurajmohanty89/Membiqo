# AI Coding Agent Instructions — Membiqo

You are implementing **Membiqo**, a multi-tenant membership SaaS. **Read this file and linked documents before writing code**. Treat `decisions.md` as canonical and `prd.md` as scope definition.

## Absolute rules

1. Brand is **Membiqo**, tagline **One Platform. Every Membership.** Do not write GymSphere anywhere in product UI.
2. Exhibition SaaS subscription price is **₹0/month**; no gateway, card collection, simulated platform checkout, or automatic billing. Future ₹99/month is **planned**, not charged.
3. Business membership fees are independent of Membiqo SaaS fees. Support ₹0 plans and optional business-owned UPI QR/cash **manual** verification; never auto-approve a paid order based on a screenshot.
4. Six launch categories use a **single shared schema and components** with labels/presets. Do not fork the app by category.
5. Treat `business_id` as a tenant security boundary in every query/mutation. Database RLS **plus** server authorization, not just frontend hiding.
6. Avoid sensitive data collection: no Aadhaar, DOB, blood group or medical fields in MVP. Use synthetic exhibition data.
7. QR attendance challenges expire after 60 seconds; server verifies identity, business, active membership, duplicate/day check.
8. Member PDF card public verification exposes only limited validity and business name. Private documents remain private.
9. Write database changes as reviewed migrations. Add tests for every authorization check, monetary state change and date boundary.
10. Implement only **one issue/milestone at a time**. Do not replace a secure requirement with a TODO that silently passes.

## Required reading order

1. `decisions.md`
2. `prd.md`
3. `architecture.md`
4. `database.md`
5. `api.md`
6. `security.md`
7. `roadmap.md`
8. `testing.md`
9. `ux.md`
10. `deployment.md`

## Engineering conventions

- Strict TypeScript, Zod validation, meaningful error codes, no `any` without justification.
- Monetary values in **integer paise**, UTC timestamps internally, IANA business timezones for display and calendar additions.
- Choose domain modules/services rather than placing business rules in React components.
- Route handlers must derive user identity from server session; no trust in client `userId` or role.
- Prefer Supabase user-scoped client with RLS; restrict service-role use to narrowly guarded server functions.
- Database constraints prevent duplicate membership activation, cross-tenant FKs, and duplicate daily check-ins.
- All sensitive storage is private. Signed URLs issued only after server authorization.
- UI must be accessible and mobile responsive. Add loading/error/empty states.
- Never put real secrets, personal records, or screenshots in commits, logs or demo fixtures.

## Definition of a finished task

Provide changed files, migration details, test results, validation commands, role/tenant threat checks and any missing items. If a spec conflicts with another, quote the two sections and propose a change to `decisions.md` rather than silently deciding.

## First task to implement

Start with **Milestone 0** in `roadmap.md`. Scaffold project and tooling, establish conventions and create brand landing shell. Do not implement payment gateway or all remaining milestones at once.
