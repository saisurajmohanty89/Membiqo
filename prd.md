# PRD — Membiqo

**Version:** 1.0 (Exhibition MVP)  
**Status:** Ready for implementation planning  
**Owner:** Founder / project team  
**Brand:** Membiqo — *One Platform. Every Membership.*

## 1. Product summary

Membiqo is a free-to-use (during the exhibition beta), mobile-first web SaaS for businesses that manage recurring memberships or enrollments. Each business receives a branded portal, owner dashboard, membership plans, member registrations, manual payment confirmation, QR attendance, downloadable digital ID cards, and simple reports. A platform admin manages all tenants. All first-release business presets use a common core and single data model.

## 2. Problem and opportunity

Small gyms, yoga studios, coaching centers, libraries, and academies often use spreadsheets, paper forms, manual check-in sheets, and informal fee tracking. Owners need one simple place to see who is active, whose membership expires, who attended, and which payments were confirmed. Members need a clear way to register, view their status, access an ID card, and check in.

## 3. Goals / non-goals

**Goals (v1):**
- Business sign-up and portal setup in a guided flow with **no SaaS payment**.
- Single architecture supporting six business presets.
- Member registration, owner review, plan requests and manual fee confirmation.
- Secure digital membership card and time-limited QR attendance.
- Tenant-isolated business dashboard and simple business-level reports.
- Demonstrable end-to-end flows using synthetic exhibition demo data.

**Non-goals (v1):** Stripe/Razorpay integration, real platform billing, auto debit, WhatsApp/SMS campaigns, Aadhaar, biometric verification, seat booking, classes/batches, franchise management, native mobile apps, AI features, advanced financial accounting, and inventory.

## 4. Target users

| Persona | Need | Primary success moment |
|---|---|---|
| Independent business owner | Stop manually tracking memberships | Sees active/expiring member counts |
| Front-desk staff | Quickly approve/check in members | Successful scan, no duplicate entry |
| Member/student | Easy enrollment and proof of access | Downloads ID; checks in |
| Platform admin | Monitor platform health & abuse | Sees and can suspend a tenant |

## 5. Scope by priority

### P0 — Must work in the exhibition

**FR-01 Authentication:** Email-based signup/sign-in, password reset, logout, session handling. New business owner and member accounts use the same identity provider. Route access is role/tenant validated server-side.

**FR-02 Business onboarding:** Owner enters name, slug (unique), category preset, city/contact, timezone, logo (optional). Creates an active tenant and free business subscription record. Subsequent owner access can switch among owned businesses. No payment screen.

**FR-03 Business public portal:** `/b/[slug]` displays business name, branding, contact, and published plans. Not found/suspended businesses display appropriate safe states. Public viewers cannot see member details.

**FR-04 Member registration:** Authenticated member registers against a business, supplying name and contact details; photograph optional. Registration initially `pending`. Owner can approve/reject. One user can register separately with multiple businesses.

**FR-05 Owner-created members:** Owner/staff can create a member without a linked user account; these records are manageable but cannot log in until a secure future linking flow. A manual member can have an owner-approved plan and attendance recorded by staff (staff check-in tool can be P1).

**FR-06 Membership plans:** Owner can create/modify/archive plans with title, description, `price_paise`, `duration_count`, `duration_unit`, and visibility. Zero-price plans permitted. Existing orders retain their purchase price snapshot.

**FR-07 Membership orders:** Member selects plan, sees fee and the business's payment instructions. For ₹0 plan, no proof required. For paid plan, show business-owned UPI QR (if configured) and/or cash/offline guidance. UPI proof upload and reference are optional supporting evidence—not proof of settlement. Owner verifies using their own payment records, then approves/rejects. No automated payment processing.

**FR-08 Membership activation & expiry:** Owner approval activates membership transactionally after registration approval and payment review. Start and exclusive end dates calculated consistently. Expired memberships fail attendance. Member/owner see current and upcoming expiry. No cron dependency for correctness.

**FR-09 Digital ID card:** Authenticated approved/active member can download a branded PDF with photo, member code, plan, expiry, and verification QR. Owners can view/download their own business's approved member cards. Public verification returns limited status only.

**FR-10 Attendance:** Staff/owner dashboard produces a short-lived, rotating QR. Signed-in member scans via browser camera. Backend checks challenge expiry, tenant, member approval and active membership, and inserts one attendance record per business-local day. Clear failure messages for expired, duplicate, unauthorized or invalid code.

**FR-11 Owner dashboard:** KPIs: total members, active members, expired members, attendance today, pending registrations, pending orders, and verified member collections over a chosen date range. Lists/filter/search/export optional where uncomplicated. Values always scoped to selected business.

**FR-12 Platform admin:** Protected panel lists tenants, counts, subscriptions, recent activity, and supports suspension/reactivation with audit reason. Admin cannot simply impersonate members in the UI. Every admin mutation audited.

**FR-13 Basic polish:** Mobile-responsive UI, empty/loading/error states, actionable validation, accessible forms, navigation by role, confirmation for destructive actions.

### P1 — Nice after P0

CSV export, staff check-in for manual members, in-app notifications, renewal queue, basic support tickets, owner-configurable logo/color, onboarding tutorial, richer analytics.

### P2 — Later products

Seat allocation, coaching batches, session bookings, automated notifications, multi-branch, subscriptions with a paid ₹99 option subject to migration/notice, gateways, trainer/course modules.

## 6. Key user journeys and acceptance criteria

### Journey A — Owner onboard

**Given** a new authenticated owner, **when** they choose a business type and complete business setup, **then** a unique business is created, a `free_beta` subscription at zero paise is recorded, and they reach the owner dashboard. A duplicate slug is rejected with a suggested alternative. No checkout occurs.

### Journey B — Member enrolls and activates

**Given** an active public portal, **when** a member registers and requests a ₹0 plan, **then** their member registration is `pending`, order is `awaiting_review`, and no attendance access exists. **When** owner approves registration and order, **then** exactly one valid membership begins and the member can download their card. A paid order requires independent manual verification before approval.

### Journey C — Attendance

**Given** an approved member with active membership, **when** the member scans an unexpired challenge, **then** attendance is recorded with server time/local date, and a success indicator appears. Repeating it that day yields an explicit "already checked in" response without a second row. Expired/mis-scoped QR is rejected.

### Journey D — Expiry

**Given** a membership ending at an instant T, **when** server time reaches T, **then** attendance is rejected and the member sees expired status immediately, even if no scheduled job ran.

### Journey E — Tenant isolation

**Given** owners A and B manage different organizations, **when** A guesses B's IDs, calls APIs, or alters request bodies, **then** A sees no B records and cannot mutate them. Similarly a member can only access their own profiles, plans they are allowed to view, card and attendance.

### Journey F — Admin suspension

**Given** platform admin suspends tenant B, **then** public enrollment, owner mutations, and attendance for B are disabled. Admins can reactivate via audited action.

## 7. Core data entities

`profiles`, `businesses`, `business_users`, `business_subscriptions`, `members`, `membership_plans`, `membership_orders`, `memberships`, `attendance_challenges`, `attendance_entries`, `card_tokens`, `platform_admins`, `audit_logs`. See `database.md` for types and relationships.

## 8. Non-functional requirements

- Security: deny-by-default tenant RBAC + database RLS; private PII and uploads; audited admin actions.
- Performance targets for demo (not guaranteed SLA): typical list interactions under ~2 seconds on a stable connection; generate PDF without blocking unrelated requests; paginate lists.
- Accessibility: semantic labels, keyboard navigation, visible focus, adequate contrast, responsive mobile viewport.
- Reliability: transactional activation; idempotency/uniqueness constraints on orders/check-ins; clear error logging.
- Privacy: collect only essential data, provide consent notice, export/delete handling to be designed before production rollout. No Aadhaar in MVP.
- Observability: record safe error metadata but never log raw tokens, private document URLs, or payment screenshots.

## 9. Success metrics for exhibition

- A new demo business can be configured without developer intervention.
- A visitor can watch the complete enrollment → approval → card → QR check-in flow.
- Two different business categories function with the same codebase.
- A second business cannot read or modify the first business's data.
- 100% of the critical test cases in `testing.md` pass; no P0 bugs on demo devices.

## 10. Release constraints and open decisions

The exhibition uses synthetic records and can run without sending SMS or handling real money. Branding, any real public release, future pricing/terms, minors' data policies, email deliverability, privacy and local compliance require separate launch review. Product choices in `decisions.md` override earlier idea brainstorming.
