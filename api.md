# Membiqo — HTTP API Contract

**Convention:** Next.js route handlers under `/api`. Use JSON, Zod validation, server-derived identity, and explicit tenant authorization. Exact handler file organization may change; keep external behavior consistent. All mutation endpoints require authenticated sessions unless explicitly noted.

## 1. Shared rules

- No user identifier / role from the browser is trusted as authority.
- UUID and slug path params validated; errors are JSON: `{ "error": { "code": "...", "message": "..." } }`.
- Return 401 for unauthenticated, 403 for known no-access, 404 for inaccessible tenant-scoped objects where necessary to avoid ID enumeration, 409 for conflicts, 422 for invalid state, 429 for rate limits.
- Paginate lists with `limit` and cursor; no unbounded `select *`.
- Data amounts are integer **paise**; no float money math. Time values are ISO-8601 UTC; business timezone shown only on presentation.
- Side-effect operations (business create, order approval, check-in) must be transactional and duplicate-safe.
- Explicitly revalidate business status `active` for write operations.

## 2. Endpoint catalog

| Method | Route | Access | Purpose |
|---|---|---|---|
| GET | `/api/me` | Authenticated | Current profile, tenant roles, member enrollments |
| POST | `/api/businesses` | Authenticated | Create business + owner role + zero-cost subscription |
| GET | `/api/businesses` | Authenticated | List businesses current user manages |
| GET | `/api/businesses/:businessId` | Owner/staff | Private business configuration |
| PATCH | `/api/businesses/:businessId` | Owner | Edit business settings |
| GET | `/api/portal/:slug` | Public | Safe business display data + published plans |
| POST | `/api/portal/:slug/register` | Authenticated | Register as business member (pending) |
| GET | `/api/businesses/:businessId/members` | Owner/staff | Search/filter tenant members |
| POST | `/api/businesses/:businessId/members` | Owner/staff | Create manual member (no linked login) |
| POST | `/api/businesses/:businessId/members/:memberId/decision` | Owner/staff | Approve/reject member registration |
| GET/POST | `/api/businesses/:businessId/plans` | GET owner/staff; POST owner | View/manage plans |
| PATCH | `/api/businesses/:businessId/plans/:planId` | Owner | Update/archive plan |
| POST | `/api/businesses/:businessId/orders` | Member of tenant | Request plan; server snapshots price |
| POST | `/api/businesses/:businessId/orders/:orderId/proof` | Owning member | Submit UPI proof/reference or indicate cash |
| GET | `/api/businesses/:businessId/orders` | Owner/staff | Pending manual confirmations |
| POST | `/api/businesses/:businessId/orders/:orderId/decision` | Owner/staff | Verify/reject; approval atomically activates |
| GET | `/api/businesses/:businessId/dashboard` | Owner/staff | Tenant-scoped KPIs and pending counts |
| GET | `/api/businesses/:businessId/attendance` | Owner/staff | Daily/range attendance list |
| POST | `/api/businesses/:businessId/attendance/challenges` | Owner/staff | Create 60-second challenge; return QR payload |
| POST | `/api/attendance/check-in` | Member | Consume challenge, verify eligibility, insert record |
| GET | `/api/businesses/:businessId/members/me` | Member | Own profile, membership state and attendance summary |
| GET | `/api/businesses/:businessId/members/:memberId/card.pdf` | Self or owner/staff | Authorize & stream fresh PDF |
| GET | `/api/verify/:token` | Public | Minimal valid/invalid badge response (rate limited) |
| GET | `/api/admin/businesses` | Platform admin | Global paginated tenant list |
| POST | `/api/admin/businesses/:businessId/status` | Platform admin | Suspend/reactivate, audit reason required |
| GET | `/api/admin/overview` | Platform admin | Global aggregate metrics |

**Auth actions** may use Supabase Auth directly rather than custom `/api/auth` routes. No platform-billing checkout endpoint exists in MVP.

## 3. Representative payloads

### Create business

```http
POST /api/businesses
Content-Type: application/json

{"name":"Demo Fitness Hub","slug":"demo-fitness-hub","kind":"gym","timezone":"Asia/Kolkata","contactPhone":"+91 9000000000"}
```

**201**:

```json
{"business":{"id":"<uuid>","slug":"demo-fitness-hub","status":"active"},"subscription":{"planCode":"free_beta","monthlyPricePaise":0,"currency":"INR","status":"active"}}
```

### Create membership plan

```http
POST /api/businesses/<businessId>/plans

{"name":"Monthly","description":"Open gym access","durationCount":1,"durationUnit":"month","pricePaise":0,"isPublished":true}
```

### Member plan order

```http
POST /api/businesses/<businessId>/orders

{"planId":"<planUuid>","paymentMethod":"free"}
```

Server retrieves current plan price. If price > 0, `paymentMethod: free` is rejected. If price = 0, order becomes `awaiting_review`; member must be `approved` before owner can approve the order.

### Submit manual UPI proof

```http
POST /api/businesses/<businessId>/orders/<orderId>/proof

{"paymentMethod":"upi_manual","paymentReference":"UPI-REFERENCE-EXAMPLE","paymentProofPath":"private/uploads/example.png"}
```

The path is accepted only if server verifies object ownership, MIME/type/size and that the uploader actually controls that object. Prefer **signed upload URLs** obtained through a separate authenticated upload route, rather than trusting an arbitrary path. For cash, staff marks receipt through an authenticated owner action; a member cannot certify that the gym received cash.

### Owner order decision

```http
POST /api/businesses/<businessId>/orders/<orderId>/decision

{"decision":"approve","note":"Checked against business account statement"}
```

**200**:

```json
{"order":{"id":"<uuid>","status":"approved"},"membership":{"id":"<uuid>","startsAt":"2026-11-16T10:00:00Z","endsAt":"2026-12-16T10:00:00Z"}}
```

Repeat approval must **not** create another membership. Approve action must refuse pending registrations and any existing overlapping active membership.

### Attendance check-in

```http
POST /api/attendance/check-in

{"challengeToken":"<opaque-random-token>"}
```

Backend hashes token, checks expiry and parent business, uses signed-in user's tenant member record, verifies membership from UTC server clock, computes `local_date` using the tenant timezone, inserts attendance entry. Do not accept client-provided `memberId`, `businessId`, `timestamp`, or `localDate` as authority.

**200**:

```json
{"status":"checked_in","checkedInAt":"2026-11-16T10:05:00Z","localDate":"2026-11-16","businessName":"Demo Fitness Hub"}
```

**409** on duplicate: `{"error":{"code":"ALREADY_CHECKED_IN","message":"You already checked in today."}}`.

## 4. Public verification

`GET /api/verify/:token` returns only:

```json
{"valid":true,"businessName":"Demo Fitness Hub","cardStatus":"active"}
```

No phone number, email, photo, medical information, full profile, raw business member ID, or payment information. Validate token format and rate limit. `valid` must reflect current membership state; expired or revoked cards fail immediately.

## 5. Required security tests by endpoint

For every business-scoped route, verify cross-tenant IDs return denied/no data; unauthorized user cannot approve/modify/issue challenges. For uploads, verify content type, size, private bucket, path ownership. For sensitive changes, verify `audit_logs` exists. See `testing.md`.
