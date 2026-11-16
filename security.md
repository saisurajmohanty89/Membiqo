# Membiqo — Security & Privacy Baseline

This is a development checklist, not a legal certification. A separate review is required before collecting real customer data or launching commercially.

## 1. Data minimization

Collect member full name, a safe contact channel, optional photo, and business-member relationship. Emergency contact is optional and should be justified. **Do not request or store Aadhaar, blood group, health records, government documents, or full DOB in v1.** Coaching centers may involve minors; use synthetic adult demo profiles during exhibition and obtain appropriate privacy/legal review before collecting children's data.

No personal contact or health information is shown in public ID verification. Member PDF cards must not be automatically public.

## 2. Threat model

| Threat | Risk | Mitigation |
|---|---|---|
| Tenant A guesses tenant B IDs | Cross-business data breach | RLS, scoped queries, composite FKs, integration tests |
| Browser changes `role` or `businessId` | Privilege escalation | Session-based identity, server RBAC, no client-trusted roles |
| Leaked service-role key | Full DB bypass | Server-only storage, least access, key rotation |
| Screenshot shared as payment proof | False activation | Owner verifies actual receipt; screenshots are evidence only |
| Static/shared attendance QR | Remote fake check-in | 60-second random challenge + auth, replay controls, rate limiting |
| ID QR reveals member details | Privacy loss | Opaque token, limited public verification output |
| Unchecked file upload | Malware / PII leak | Private storage, size/type/signature validation, restricted file formats |
| Fake admin claims | Whole-platform compromise | Privileged server check, MFA for admins, auditable actions |
| Automated registrations / token guesses | Spam / abuse | Rate limits, CAPTCHA where needed, random tokens, throttling |
| Expired but still cached membership | Unauthorized access | Validate current server time on every check-in and card verification |

## 3. Minimum controls before exhibition

- HTTPS everywhere outside local development.
- Supabase Auth configured with secure cookie/session management; never store passwords in custom tables.
- RLS enabled on all relevant tables and tested using ordinary user JWTs; anon key alone is **not** a permission boundary.
- Strong random tokens: QR challenge and public card token generated with cryptographically secure randomness. Persist hashes (SHA-256 or secure keyed digest) rather than plaintext.
- Business status `suspended` blocks tenant owner mutations, member enrollment/activation and check-in.
- Role matrix enforced in server API and DB policies; admin paths check privileged server data.
- No service-role secret in browser bundle, logs, public environment variables, or commits.
- Uploads stay in private buckets; server issues short-lived signed links only after authorization.
- Limit file uploads (for example: 5 MB photos, 5 MB proof), validate MIME plus signature, block SVG/scriptable content unless sanitized; avoid displaying untrusted files inline.
- Rate limit login, registration, QR challenge issuance, check-in and verification token queries; use short generic errors to reduce enumeration.
- CSRF-aware same-site session protections; validate origin/host for cookie-authenticated writes as needed.
- Database backups, migration history, sanitized error logging; test restore procedure before broader launch.
- Admin actions (suspend/reactivate) and payment approval decisions recorded in append-only audit trail.

## 4. Role permission matrix

| Action | Platform admin | Owner | Staff | Member | Visitor |
|---|---|---|---|---|---|
| View public business portal | Yes | Yes | Yes | Yes | Yes |
| Manage business settings and plans | Support oversight only | Yes | Limited/no plan editing | No | No |
| Review member registrations | Via audited exceptional support only | Yes | Yes | No | No |
| Confirm offline member payment | No routine action | Yes | Yes | No | No |
| View tenant member list | Exceptional audited support only | Yes | Yes | Self only | No |
| Generate business attendance QR | No | Yes | Yes | No | No |
| Scan/check in as member | Only when personally enrolled | If personally enrolled | If personally enrolled | Yes | No |
| View private card | With explicit audited support workflow only | Tenant members | Tenant members | Own only | No |
| Suspend/reactivate business | Yes | No | No | No | No |

**Platform admin scope is not automatic carte blanche to access members' personal files.** Prefer aggregate metrics and exceptional, logged support access with a reason.

## 5. QR token design

- Staff authenticated request returns a random QR challenge token, its TTL (60 seconds) and display payload.
- Server stores hash + associated business + expiration; client sees plaintext token only briefly.
- QR payload should be a deep link or encoded token; never embed credentials, member personal data, or an API secret.
- A scan is valid only if the user's membership is approved and active for the **challenge's** business. Reject wrong-tenant, expired, malformed, and rate-limited requests.
- At most one check-in per member/business/local date. Challenge can be redeemed by multiple legitimate members during its TTL; it is not single-use globally.
- This prevents long-term reuse but does **not** perfectly prevent a live QR being relayed remotely. Later anti-fraud options: supervised scan, kiosk, device/location check with appropriate consent.

## 6. Privacy and operational practices

- Publish clear privacy information before collecting real member data; identify business vs Membiqo responsibilities, retention, deletion and contact process.
- Build an authorized account/data deletion path before public production use, considering records that must be retained for legitimate audit purposes.
- Set retention policy for payment screenshots and QR challenges; do not keep proof forever without a reason.
- All demo accounts, names, phones, screenshots and cash entries should be fictional and conspicuously marked demo data.
- Do not promise legal compliance based on this document alone. Have Indian privacy/payment requirements reviewed at commercial launch.

## 7. Security release blockers

**Do not deploy publicly** if any test demonstrates cross-tenant access, owner/member role escalation, client forgery of attendance, unprotected admin routes, publicly readable payment proofs, exposed service key, or double activation of a single order.
