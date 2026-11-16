# Membiqo — UX / Screens / Design System

## 1. Brand direction

**Name:** Membiqo  
**Tagline:** One Platform. Every Membership.  
**Personality:** Modern, trustworthy, easy for a small-business owner, not corporate-heavy.  
**Style:** Clean SaaS UI, spacious cards, rounded corners, clear tables, mobile-first navigation.

Suggested initial palette (modifiable): deep navy `#10233E`, electric indigo `#5459E8`, turquoise accent `#13B9A6`, neutral surfaces `#F7F9FC`, accessible text `#172234`. Use semantic design tokens and verify contrast; don't encode colors in business logic. Font: a legible system sans or a locally hosted licensed sans font.

## 2. Route map

```text
/                             Landing: benefits, demos, free pricing
/pricing                      ₹0 exhibition beta; later ₹99 teaser clearly future-only
/login  /signup  /forgot-password
/onboarding                   Business creation wizard
/b/[slug]                     Public business portal + plan catalog
/b/[slug]/join                Sign in/up, member registration, request plan
/dashboard/[businessId]       Owner overview
/dashboard/[businessId]/members
/dashboard/[businessId]/plans
/dashboard/[businessId]/orders
/dashboard/[businessId]/attendance
/dashboard/[businessId]/settings
/member/[businessId]          Member overview, status, ID, check-in
/member/[businessId]/card
/verify/[token]               Minimal public verification view
/admin                        Restricted platform admin dashboard
/admin/businesses
```

Route groups/layouts may be used; paths above are public URL intent. API routes are listed in `api.md`.

## 3. Site map by role

- **Visitor:** Home → Pricing (free) → Create account → Create business; or business portal → Sign up to join.
- **Owner:** Overview → Members → Plans → Orders & Verification → Attendance (display rotating QR) → Reports → Settings.
- **Staff:** Members, orders verification, attendance QR, attendance history; no business ownership changes.
- **Member:** My membership → Plan request status → Digital ID PDF → Scan attendance QR → Attendance history.
- **Admin:** Platform overview → Businesses → Subscription status → Suspension/reactivation audit.

## 4. Onboarding details

Owner wizard: (1) account, (2) business name and category preset, (3) slug and contact, (4) choose timezone, (5) dashboard. Show **Free forever during exhibition beta / current ₹0 launch** language; avoid falsely guaranteeing permanent free use if future pricing is planned. No card required.

Business type preset controls copy:

| Preset | Member label | Plan label | Dashboard hint |
|---|---|---|---|
| Gym / Yoga / Martial Arts | Member | Membership plan | Manage memberships and check-ins |
| Coaching Center | Student | Fee plan | Track student enrollment and attendance |
| Study Center | Subscriber | Access plan | Track subscriptions and visits |
| Sports Academy | Athlete | Training plan | Track athletes and membership status |

Never fork all routes for each label variant. Use a presentation config object and accessible text.

## 5. Required states on every screen

Empty list, loading/skeleton, server error with retry, no permission, suspended business, validation error, pending approval, active, expiring soon (e.g., under 7 days), expired, already checked in, QR expired, camera permission denied/unavailable. Offer a non-camera fallback only if staff can securely verify in person.

## 6. Form specifications

### Business form

Required: business name, slug, category, timezone. Optional: logo, email, contact phone, city, social links, business UPI QR upload. Validate slug and unique availability server-side; sanitize displayed text.

### Member registration

Required: full name and one contact channel (verified account email or contact phone, as appropriate). Optional: photo, emergency contact. Do **not** collect Aadhaar, blood group or full DOB. Include clear privacy copy and consent for optional fields.

### Plan form

Name, description, price in INR (UI), duration count, duration unit, published toggle. Convert rupees to integer paise in server validation. Show examples: 1 day, 1 month, 3 months, 1 year; category-specific copy only.

### Manual payment review

Member view distinguishes **No fee (₹0)**, **UPI paid to business—awaiting confirmation**, and **Cash/office payment—pending business confirmation**. Owner sees reference, evidence (if any), plan/amount, and an explicit "I verified receipt" action. Do not label screenshot upload "payment successful".

## 7. Card layout

Card front: business logo/name, optional member photo, member name, member code, plan, start date, expiry date, status, verification QR. Responsive preview and printable PDF. Public `/verify` should show only "Valid / Not valid" and business name, not profile details.

## 8. Exhibition demo script

1. Open home and show multi-industry tiles and ₹0 free plan.
2. Register a **Gym** business, create a free 1-month plan.
3. Register fictional member in its portal.
4. Log in as owner to approve member and order.
5. Download a digital ID and open its verification URL.
6. Display rotating QR from owner attendance page; member scans and checks in.
7. Try scanning twice; show duplicate prevention.
8. Switch to a seeded **Coaching** business and show same engine, different terminology.
9. Show separate tenant records and admin overview.

Never use real personal IDs, actual member payment screenshots, or real bank credentials in a demo.
