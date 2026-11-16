# Membiqo — Test Plan & Acceptance Matrix

**Recommended tooling:** Vitest for domain functions, React Testing Library for UI, Playwright for browser E2E, integration tests with a real disposable PostgreSQL/Supabase environment and RLS enabled. Do not mock RLS when validating tenant isolation.

## 1. Testing layers

1. **Unit:** membership calendar arithmetic, state derivation, access decision helpers, price conversions, input validation, token hashing.
2. **Integration:** database foreign keys/RLS, transactional activation, role changes, storage access, duplicate attendance, suspended businesses.
3. **API:** authenticated/unauthenticated requests, tampering, error consistency, pagination, file upload checks.
4. **E2E:** owner/member/admin workflows with synthetic accounts and mobile viewport.
5. **Manual exhibition rehearsal:** poor network, camera permissions, projector QR brightness, mobile browser support.

## 2. P0 test cases

| ID | Scenario | Expected result |
|---|---|---|
| T01 | Owner creates a new business | Tenant, owner association, and `free_beta`/0-paise subscription exist atomically |
| T02 | Owner chooses existing slug | Validation error; no partial tenant created |
| T03 | Owner signs up with no bank/payment data | Setup succeeds; no platform checkout |
| T04 | Member self-registers | Pending registration; no access to check-in/card |
| T05 | Owner manually adds member | Member row has null `user_id`; cannot be taken over by phone match |
| T06 | Member requests a zero-price plan | Order `awaiting_review`; no uploaded proof required |
| T07 | Paid member order with only screenshot | No activation until owner confirms actual receipt |
| T08 | Approve order before member approval | Rejected; no membership created |
| T09 | Concurrent duplicate approval requests | Exactly one order approval and one membership record |
| T10 | Member scans challenge with active membership | One attendance entry in correct tenant |
| T11 | Same user scans again same business-local day | 409 already checked in; one DB row |
| T12 | Expired QR challenge | Rejected; zero attendance rows |
| T13 | Member without active membership | Rejected even if QR is fresh |
| T14 | Membership ends at exclusive `ends_at` | Attendance denied at exact end instant |
| T15 | Tenant A owner requests Tenant B members | No B data and no change permitted |
| T16 | Member A requests Member B card PDF | Forbidden/not found; no PDF bytes |
| T17 | Forged tenant ID in order / attendance body | Ignored/denied; no cross-tenant write |
| T18 | Cross-tenant foreign key attempt | Database rejects insert |
| T19 | Unauthenticated user opens `/admin` | No protected data; login/forbidden route |
| T20 | Normal owner calls admin suspension endpoint | Forbidden; no mutation |
| T21 | Admin suspends business | Enrollment, check-ins and owner mutation blocked; audit event exists |
| T22 | Public card QR when membership expires | Limited verifier returns invalid |
| T23 | Public verifier response | No PII, email, photo, phone, payment evidence |
| T24 | Attempt to fetch private payment proof without access | Forbidden or expired signed URL |
| T25 | Different business category preset | Labels differ but same workflow and tenant rules |
| T26 | Upload spoofed content / oversized file | Rejected; never publicly accessible |
| T27 | Supabase browser env | Public anon key only; service-role never shipped to client |
| T28 | Pricing page | Clearly ₹0 now, ₹99 only future; no checkout route |

## 3. Calendar-duration edge cases

| Input (business-local) | Duration | Expected exclusive end |
|---|---|---|
| 2026-06-01 10:00 | 1 month | 2026-07-01 10:00 |
| 2026-01-31 10:00 | 1 month | 2026-02-28 10:00 |
| 2028-01-31 10:00 | 1 month | 2028-02-29 10:00 |
| 2028-02-29 10:00 | 1 year | 2029-02-28 10:00 |
| 2026-06-01 10:00 | 1 day | 2026-06-02 10:00 |

Test timezone conversion for `Asia/Kolkata` and a zone with DST. Use a controllable test clock; do not introduce flaky real-time waits.

## 4. Test fixture strategy

Create synthetic users: `owner_a`, `staff_a`, `member_a`, `owner_b`, `member_b`, `platform_admin`. Businesses: a gym and a coaching center, each with unique plans/member rows. Include one pending order, one approved zero-cost order, one expired membership. No real Aadhaar, real bank screenshots or real phone numbers.

## 5. CI/release gate

- TypeScript typecheck, lint, unit tests and integration tests pass.
- API security and end-to-end P0 cases pass in staging.
- Security review verifies RLS policies, upload access, elevated credentials and admin audit.
- Basic responsive/mobile checks pass (360 px viewport minimum target).
- No unreviewed database schema changes, hardcoded secret keys, unbounded queries, or placeholder payment gateway UI.
- Demo rehearsal works even if notification sending is disabled.
