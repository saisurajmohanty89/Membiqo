# Membiqo 🚀

**One Platform. Every Membership.**

Membiqo is a **multi-tenant, mobile-first membership-management SaaS** for small organizations: gyms, yoga studios, martial-arts academies, coaching centers, libraries/study centers, and sports academies. Businesses create their own branded member portal; members register, request a plan, receive digital ID cards after activation, and check in using a time-limited QR code. Owners manage members, plans, approvals, attendance, and reports. Platform administrators manage the overall service.

> **Release policy:** The exhibition / public beta is **₹0/month for every business**. No payment gateway or platform charge in v1. A **possible future ₹99/month business subscription** is a *planned product change*, **not an active charge**. Member fees, if a business chooses to charge them, remain between that business and its members (cash/offline or a business-owned UPI QR with owner verification). Do not confuse SaaS fees with the business's membership fees.

## Read this first

| Document | Purpose |
|---|---|
| [prd.md](prd.md) | Product scope, personas, requirements, and acceptance criteria |
| [decisions.md](decisions.md) | Canonical product decisions and definitions; resolves ambiguity |
| [architecture.md](architecture.md) | Technical design, boundaries, tenancy, and request flows |
| [database.md](database.md) | Proposed PostgreSQL schema, constraints, tenancy, and RLS |
| [api.md](api.md) | HTTP endpoints, request/response shapes, and access rules |
| [security.md](security.md) | Threat model, privacy, permissions, QR safeguards |
| [ux.md](ux.md) | Routes, user journeys, design direction, and forms |
| [roadmap.md](roadmap.md) | AI-friendly, sequenced implementation tasks and milestones |
| [testing.md](testing.md) | Unit, integration, E2E, and tenant-isolation test cases |
| [deployment.md](deployment.md) | Environment, setup, migrations, demo seed, and deployment |
| [ai-prompts.md](ai-prompts.md) | Copy/paste prompts for AI coding assistants |
| [AGENTS.md](AGENTS.md) | Repository rules for AI coding agents |
| [.env.example](.env.example) | Environment variable placeholders (never actual credentials) |

## Tech stack (recommended baseline)

- **Web:** Next.js App Router, React, TypeScript, Tailwind CSS, shadcn/ui.
- **Identity, data & storage:** Supabase Auth, PostgreSQL, Row Level Security (RLS), private Supabase Storage buckets.
- **Validation:** Zod, plus server-side authorization on every mutation.
- **PDF:** `@react-pdf/renderer` or a supported server-side PDF package (pick one, test server deployment).
- **QR:** Browser camera scanner for members, server-generated short-lived check-in challenges.
- **Monitoring and testing:** Structured error logs, Vitest, React Testing Library, Playwright.
- **Hosting:** Vercel for the web app and Supabase for the backend (or compatible deployment).

Use maintained compatible versions at implementation time; avoid hardcoding an unverified framework major version into this documentation.

## Business types supported at launch

| Preset | MVP behavior | Later module |
|---|---|---|
| Gym | Members, monthly plans, QR attendance | Trainers/workouts |
| Yoga studio | Memberships, QR attendance | Class bookings |
| Martial arts academy | Memberships, QR attendance | Ranks/belts |
| Coaching center | Students (members), fees, QR attendance | Batches/timetables |
| Library/study center | Memberships, QR attendance | Seats/shifts |
| Sports academy | Members, fees, QR attendance | Sessions/coaches |

All six presets use **one common product and schema**. Presets initially change labels, icons, default plan suggestions, and landing copy—not core data ownership rules. Avoid six independently coded dashboards.

## Key roles

1. **Platform admin** – oversee businesses, suspend accounts, review aggregate metrics; privileged access is server-only, audited.
2. **Business owner** – manage one or more businesses, plans, members, payments, attendance, reports.
3. **Business staff** – limited business-scoped operational actions, no billing or ownership changes.
4. **Member** – can access only their own records for each business they join.
5. **Visitor** – public landing and business portal; may initiate registration.

## MVP happy path

1. Business owner signs up and creates an organization with a unique slug, e.g. `/b/fit-hub`.
2. A free business subscription is provisioned at ₹0/month; no checkout screen.
3. Owner creates a membership plan (including a **₹0** plan if desired).
4. Member opens the portal, authenticates, registers, and requests a plan.
5. If paid offline/UPI, owner verifies the payment; if free, no payment proof is needed. Owner approves the registration and the plan request.
6. System activates a time-bounded membership and makes a PDF ID card available.
7. Staff displays a rotating attendance QR; the member scans it while signed in.
8. Server checks the user's business membership and current validity; attendance is saved once per local day.
9. Owner sees members, attendance, expiry lists, and verified collection totals.

## Suggested repository layout

```text
membiqo/
├─ app/
│  ├─ (marketing)/                 # Landing, pricing, contact
│  ├─ (auth)/                      # Sign-in, sign-up, recovery
│  ├─ (owner)/dashboard/[businessId]/
│  ├─ (member)/member/[businessId]/
│  ├─ b/[slug]/                    # Public business portal
│  ├─ verify/[token]/              # Public minimal ID verification
│  ├─ admin/                       # Protected platform admin
│  └─ api/                         # Route handlers
├─ components/                     # UI and shared components
├─ lib/
│  ├─ auth/                        # Auth / session utilities
│  ├─ permissions/                 # RBAC and tenant guards
│  ├─ billing/                     # Platform free plan abstraction
│  ├─ membership/                  # Expiry & activation rules
│  ├─ attendance/                  # Short-lived QR challenge logic
│  ├─ db/                          # Queries and repositories
│  └─ validators/                  # Zod schemas
├─ supabase/
│  ├─ migrations/                  # Versioned SQL
│  └─ seed.sql                     # Synthetic development data only
├─ tests/
│  ├─ unit/
│  ├─ integration/
│  └─ e2e/
├─ public/
├─ AGENTS.md
├─ prd.md
├─ architecture.md
└─ ...
```

## Start building with an AI coding agent

1. Place these files at the root of an empty repository.
2. Ask the agent to read `AGENTS.md`, `decisions.md`, `prd.md`, `architecture.md`, and `database.md` **before editing code**.
3. Copy the **Bootstrap prompt** in `ai-prompts.md`.
4. Have the agent implement **one milestone at a time** from `roadmap.md`; demand migrations, tests, and a checkpoint summary for each milestone.
5. Configure `.env.local` from `.env.example`. Never check real keys into Git.
6. Test two separately owned businesses to verify isolation before deploying.

## MVP definition of done

A new business can set up its portal for **₹0**, create plans, onboard members, approve purchases, issue cards, record secure QR attendance, and view tenant-scoped reports. Platform admin can administer businesses. Core journeys work on mobile. No real gateway, no mandatory Aadhaar, no unrestricted cross-business access. Critical tests pass. See `testing.md`.

## Useful official docs

- [Next.js App Router](https://nextjs.org/docs/app)
- [Supabase Auth](https://supabase.com/docs/guides/auth)
- [Supabase PostgreSQL RLS](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supabase Storage](https://supabase.com/docs/guides/storage)

**Status:** Planning specification. The documents are implementation instructions, not a claim that the application is already built.
