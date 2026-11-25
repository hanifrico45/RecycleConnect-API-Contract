# API Guide

## Base path and format

The backend mounts application routes beneath `/v1`. Send and receive JSON with `Content-Type: application/json`. The local server defaults to `http://localhost:3000`; configure a frontend base URL per environment. The PRD's production URL (`https://api.recycleconnect.ng/v1`) is a target, not a verified live service.

The root health endpoints are outside the versioned API: `GET /` returns `{ "app": "RecycleConnect API", "status": "running", "docs": "See README.md" }`; `GET /health` returns `{ "ok": true }`.

## Supabase configuration status

Supabase hosts PostgreSQL for this project. The password-auth migration was applied manually, but the local backend still needs a private `DATABASE_URL`. The backend uses the `pg` driver and reads `DATABASE_URL`; keep the connection string in the backend environment. Keep it in the backend environment, never in frontend code or a committed `.env` file. The backend copy contains a blank `.env.example` and ignores `.env`.

`SUPABASE_URL` and `SUPABASE_PUBLISHABLE_KEY` are reserved placeholders only. The current auth routes do not use Supabase Auth: Google sign-in and email/password routes issue the backend's own JWT. If the team chooses Supabase Auth, the frontend sign-in flow and backend token verification must be updated together. An API URL or publishable key alone does not connect the existing `pg` database client.

## Authentication and permissions

`POST /v1/auth/google` is public. The client sends a Google `id_token`; on a new account it also sends a role. The backend verifies the token using `GOOGLE_CLIENT_ID`, creates or finds a user, then returns a JWT valid for seven days. The token payload contains the user ID and role. Send it on protected routes as `Authorization: Bearer <JWT>`.

Roles are `HOUSEHOLD`, `ESTATE`, `COLLECTOR`, and `ADMIN`. Household and estate users share onboarding and pickup operations. Collector routes require `COLLECTOR`; map, status, and weigh routes additionally require approved collector KYC. Admin routes require `ADMIN`. The Google signup route permits new `HOUSEHOLD`, `ESTATE`, and `COLLECTOR` accounts; it does not create a new admin account.

`POST /v1/auth/otp/verify` requires a collector JWT. The OTP is hard-coded to `123456` in the current backend and is a development/demo behavior, not a real verification service.

## Error responses

Errors are JSON objects with a single `error` string. The app maps malformed JSON to `400`, explicit validation errors to their route's status, missing/invalid/expired JWTs to `401`, role or KYC failures to `403`, missing records or routes to `404`, invalid state transitions to `409`, and unhandled server errors to `500` with `Something went wrong on the server`. Do not rely on database-specific error text.

Examples:

```json
{ "error": "Missing token. Send Authorization: Bearer <JWT>" }
```

```json
{ "error": "Access denied. Allowed roles: HOUSEHOLD, ESTATE" }
```

## Implemented route inventory

All versioned paths below are under `/v1`. Exact request and response schemas are in [`openapi.yaml`](openapi.yaml).

| Method and path | Permission | Purpose |
| --- | --- | --- |
| `POST /auth/google` | Public | Google sign-in and account creation |
| `POST /auth/otp/verify` | Collector JWT | Verify demo OTP and save phone |
| `PATCH /users/onboarding` | Household, Estate | Save profile and create the first recurring pickup |
| `PATCH /users/pause-toggle` | Household, Estate | Pause/resume recurring pickups |
| `GET /rates` | Any authenticated role | Read material rates |
| `GET /pickups/dashboard` | Household, Estate | Read dashboard, wallet, rates, history, notifications |
| `POST /pickups/one-off` | Household, Estate | Schedule a one-off pickup |
| `PATCH /pickups/{id}/skip` | Household, Estate | Skip a scheduled pickup slot |
| `POST /pickups/{id}/confirm` | Household, Estate | Confirm a completed pickup or open a dispute |
| `POST /collectors/kyc` | Collector | Submit KYC details after phone verification |
| `GET /collector/map-pins` | Approved Collector | List eligible pickup pins, optionally by date |
| `PATCH /collector/pickups/{id}/status` | Approved Collector | Move a pickup to en route or arrived |
| `POST /collector/pickups/{id}/weigh` | Approved Collector | Record weights and credit household wallet |
| `PATCH /admin/rates` | Admin | Update material rates |
| `GET /admin/collectors/pending` | Admin | List collectors awaiting review |
| `PATCH /admin/collectors/{id}/verify` | Admin | Approve or reject collector KYC |
| `GET /admin/overview` | Admin | Read daily totals and disputes |
| `GET /admin/disputes` | Admin | List disputes |
| `PATCH /admin/disputes/{id}/resolve` | Admin | Resolve a dispute |

### Prepared backend additions

These routes are available in GitHub `Production` as of commit `dd0be3e`. They are included in the OpenAPI file and ready for frontend integration.

| Method and path | Permission | Purpose |
| --- | --- | --- |
| `POST /auth/signup` | Public | Email/password registration |
| `POST /auth/login` | Public | Email or phone/password login |
| `PATCH /users/profile` | Household, Estate | Update profile without rebuilding recurring pickups |

## Expected user flows

1. A user signs in with Google, chooses an allowed account role, then saves a household/estate profile with `PATCH /users/onboarding`.
2. Household/estate screens call `GET /pickups/dashboard` and `GET /rates`; users can pause, skip one scheduled slot, or schedule a one-off pickup.
3. A collector signs in, verifies the demo phone OTP, submits KYC, and waits for an admin decision. Approved collectors can view map pins, update pickup status, and submit weights.
4. Weighing credits the household wallet before user confirmation. The household then sends `THUMBS_UP` or `THUMBS_DOWN`; a thumbs-down creates a dispute for admin review.
5. Admins review KYC, change rates, inspect daily totals/disputes, and resolve disputes.

Frontend `main` commit `b203df9` calls the documented APIs through a shared browser client. The household pages and collector/admin web workspaces are connected to their corresponding routes. This does not provide the PRD's native Android collector app.

## Behavior and scope caveats

- `GET /collector/map-pins` currently includes pickup materials, address, date, and status in addition to coordinates and ID. The MVP brief says map pins should not reveal materials or weights. Backend behavior must be corrected or the brief owner must explicitly revise that rule before clients rely on pin contents.
- The weigh endpoint requires a `scale_photo_url` when the householder is absent, but it cannot prove that the image was captured live or by a camera. No upload endpoint or signed-upload contract exists in the repository. The PRD's camera-only and Cloudinary ≤300 KB rules are not enforced by this API.
- Starting a route stores a notification message containing collector name and badge; a collector photo is not part of that notification response. Dashboard data includes the badge, but not a photo URL.
- Weighing returns integer Naira payout and wallet balance is represented as Naira. The PRD equates one point to one Naira, but the backend has no explicit points field, points ledger, or redemption endpoint. Payment-provider cash-out is explicitly post-MVP.
- Payout code rounds each material's `weight * rate` before summing. The PRD describes the sum of each weight times its rate; the rounding rule should be agreed and documented before financial use.
- One-off pickup creation requires a future date and `load_photo_url`; it does not accept an address because the backend associates the pickup with the authenticated user's saved profile.
- The backend repository has no database schema or seed script checked in. The API assumes tables such as `users`, `pickups`, `material_rates`, `collector_profiles`, `wallet_transactions`, `notifications`, and `disputes` already exist.

## Team decisions still needed

- Confirm the backend database DDL and initial material rates; align it with the PRD schema and response fields.
- Confirm the production/staging URLs and release branch. The PRD names Render/Railway and a production URL, but repository configuration does not establish a deployed service.
- Confirm email/password signup remains in scope. The MVP source specifies Google sign-in; email/password was added separately by request (see [`BACKEND-PARITY.md`](BACKEND-PARITY.md)).
- Resolve the map privacy mismatch, live-camera enforcement, scale-photo storage/upload ownership, notification collector-photo requirement, and payout rounding rule.
- Confirm whether frontend/backend teams will use the exact OpenAPI response for the raw `user` database row; the database schema is not checked into the backend repository.


