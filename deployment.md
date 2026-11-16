# Membiqo — Local Setup, Configuration & Deployment

This is the planned setup for an **implementation that does not exist yet**. Do not treat example commands as proof that a repository is already runnable.

## 1. Prerequisites

- Current supported Node.js LTS and package manager (`npm` is used below).
- Supabase project or local Supabase CLI / PostgreSQL instance.
- Hosting account supporting current Next.js server features.
- Test mailbox accounts for synthetic users; no real payments or government documents.

## 2. Project setup (once source code is generated)

```bash
npm install
cp .env.example .env.local
# Add your own legitimate project credentials to .env.local (never commit it).
npm run dev
```

Expected scripts after Milestone 0: `dev`, `build`, `start`, `lint`, `typecheck`, `test`, `test:e2e`. Agent should define and verify each script rather than assuming it exists.

## 3. Configuration

See `.env.example`. Public URL and **publishable/anon** Supabase key are browser-safe only when RLS is configured correctly. The service-role key is secret and must never have a `NEXT_PUBLIC_` prefix. Keep it server-only and prefer avoiding its use for normal tenant requests.

Use separate environments for development, staging, and exhibition/production. Set allowed auth redirect URLs and site URL correctly for each. Keep backups and access controls in place.

## 4. Database migrations

- Generate migrations for all tables, foreign keys, helper functions, RLS policies and indexes.
- Apply migrations in staging, run security/integration tests, then apply to production with backup/rollback procedure.
- Never manually bypass RLS as a workaround for a failing test.
- Make `free_beta`, `monthly_price_paise=0`, `currency=INR` the new-tenant default.

## 5. Seed data for the exhibition

Provide a **development-only** `seed` script that creates:

- Gym tenant: `demo-fitness-hub` with a 1-month ₹0 plan, two fictional members (one active, one pending).
- Coaching tenant: `bright-learning-demo` with one free plan and one expired fictional member.
- Separate owners, one staff user, member accounts, and privileged demo admin created via controlled local setup.
- Example attendance entry and pending manual-payment order clearly labeled simulated.

Never seed real banking screenshots, Aadhaar or PII. Disable or limit demo accounts outside exhibition; use unique passwords and rotate them after demonstration.

## 6. Deployment checklist

- [ ] Environment variables present and scoped correctly.
- [ ] HTTPS and allowed auth redirect URLs configured.
- [ ] Production migrations applied and RLS checks passed.
- [ ] Server-only secrets not present in frontend JavaScript bundle.
- [ ] Private storage buckets and upload restrictions verified.
- [ ] Business slug and public portal functioning.
- [ ] Platform admin protected with strong authentication; MFA where available.
- [ ] Backup / restore path documented.
- [ ] Error monitoring enabled without sensitive logging.
- [ ] Phone camera scanning tested on supported browsers and HTTPS.
- [ ] QR rotation tested on screen/projector with correct time synchronization.
- [ ] All critical cases in `testing.md` passed.
- [ ] `/pricing` clearly states ₹0 for the beta, future ₹99 only a roadmap idea.
- [ ] No active real payment gateway or platform checkout.

## 7. Future ₹99 plan rollout

**Do not automatically move existing users to paid billing.** Later, ship a new PRD and migration containing plan versioning, pricing page update, customer notice, opt-in and terms, dunning/access policy, gateway provider decision and financial reconciliation. Preserve free subscriptions appropriately during transition.

## 8. Operational runbook

If sign-in fails: check Supabase Auth settings, redirect URLs and project status. If owner sees zero data: check tenant join and RLS, not `service_role` bypass. If scan fails: check browser camera permission, challenge TTL/time sync, member status and membership dates. If card PDF fails: inspect server error logs for rendering, verify actor ownership and token issuance; never make private data public as a workaround.
