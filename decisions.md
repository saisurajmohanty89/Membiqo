# Membiqo — Canonical Decisions (v1)

This is the **single source of truth for cross-document decisions**. If a task appears ambiguous, preserve these choices and document any proposed change before implementing it.

| ID | Decision |
|---|---|
| D01 | Product name is **Membiqo**; tagline **One Platform. Every Membership.** Do not use GymSphere in UI, URLs, or code identifiers. |
| D02 | Exhibition beta SaaS pricing is **₹0/month** for every registered business. Do **not** build or simulate a paid gateway or ask for credit-card data. |
| D03 | **₹99/month** is a *future possible platform plan*, not a price charged in v1; retain a plan abstraction with `free_beta` now. Migration to paid requires a new release, clear notice, and explicit opt-in/terms—not an automatic surprise charge. |
| D04 | A business may independently create **free or paid member plans**. Member payments are **cash/offline or business-owned UPI QR + manual verification**, and money goes directly to the business. Membiqo never receives or processes this money in v1. |
| D05 | Initial business types: `gym`, `yoga_studio`, `martial_arts`, `coaching_center`, `study_center`, `sports_academy`. They share tables, workflows, and core dashboard; category differences are presentation presets. |
| D06 | Tenant = business. All business-owned records are scoped by `business_id`. Owners may own multiple businesses; members may join multiple businesses via distinct member records. |
| D07 | Member registration and plan requests are separate approvals. A paid plan becomes active only after registration approval and manual payment confirmation. A ₹0 plan may auto-pass payment validation, but still requires owner registration approval by default. |
| D08 | Active membership = `starts_at <= current instant < ends_at` and `cancelled_at IS NULL`; expiry is **derived**, never based solely on an asynchronous job or mutable status flag. |
| D09 | Membership durations use `duration_unit` (`day`, `month`, `year`) and `duration_count`; month/year math uses *calendar arithmetic clamped to the last valid day* in the business timezone. End instant is exclusive. |
| D10 | QR attendance uses **server-issued, short-lived (60-second) challenge tokens** shown on a staff/owner device. Member must be logged in, active, and approved. Duplicate check-ins for the same member and business-local date are rejected safely. Printed permanent QR-only check-in is not secure and is out of MVP. |
| D11 | The downloadable member ID PDF includes business brand, member name, code, photo (if provided), plan, start/expiry, and a verification QR. Public verification displays only validity and minimal business identification. No personal contact details or health data on public verification. |
| D12 | **Do not collect Aadhaar** or other government ID uploads in the MVP. DOB, health information and blood group are not required. Emergency contact is optional, if needed and consented to. For exhibition demo, use fictional/synthetic people. |
| D13 | Platform admin privileges are server-verified, audited, and not inferred from a client-side role string. Tenant boundaries must still be respected during ordinary actions. |
| D14 | UI defaults: responsive mobile-first, accessible controls, clear empty states, display amounts in INR, business-selectable timezone (default `Asia/Kolkata`), internal timestamps in UTC. |
| D15 | The system must support both **owner-created manual member records** (not yet linked to a user) and **self-registered member accounts**. A secure claim/invite mechanism can be deferred; owner-created records cannot be self-claimed using only a phone number. |
| D16 | No real messages sent during exhibition by default. Reminder queue may be implemented later; use in-app expiry indicators in v1. |
| D17 | Member price snapshots are stored on purchase orders. A plan price edit does not retroactively alter past orders. Owner revenue charts reflect **owner-verified member collections**, not Membiqo SaaS revenue and not unverified screenshots. |
| D18 | No seat booking, batches, class schedules, trainer tools, inventory, appointment booking, or integrated payment automation in exhibition MVP. They are extensions. |

## Status glossary

- Business: `active` or `suspended`.
- Business subscription: `active`, `cancelled`; initial record `plan_code=free_beta`, `monthly_price_paise=0`, `currency=INR`, `status=active`.
- Member registration: `pending`, `approved`, `rejected`.
- Member order: `pending_payment`, `awaiting_review`, `approved`, `rejected`, `cancelled`.
  - Zero-price order enters `awaiting_review` directly.
  - For paid member plans, member chooses `upi_manual` or `cash`. For UPI, submission of a reference/proof moves it to `awaiting_review`; for cash, owner records receipt and reviews.
  - Approval of an order requires registration `approved`. It creates exactly one membership in a database transaction.
- Membership: compute `upcoming`, `active`, `expired`, `cancelled` using timestamps and cancellation fields. Do not persist a potentially stale status.

## Membership activation details

When an order is approved, set `starts_at = now()` unless an owner-scheduled future start is explicitly supported; compute `ends_at` from the plan duration and configured business timezone. Only one **currently active** membership is allowed per member per business in MVP. If the user purchases while already active, reject with a clear "renewal not yet supported" message. Extension/queuing can be a later feature.

## Demo rule

Never present a seeded payment screenshot or simulated cash transaction as a real payment. All exhibition examples must be clearly synthetic.
