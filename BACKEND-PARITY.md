# Backend Repository Parity

The API contract describes the linked GitHub backend branch `Production`. The email/password signup and login routes, profile route, and password-auth migration were pushed to that branch in commit `dd0be3e`.

## Backend additions on `Production`

| Route or file | Behavior | Status |
| --- | --- | --- |
| `POST /v1/auth/signup` | Email/password account creation | Present on `Production` (commit `dd0be3e`) |
| `POST /v1/auth/login` | Email or phone/password login | Present on `Production` (commit `dd0be3e`) |
| `PATCH /v1/users/profile` | Update profile fields without rebuilding recurring pickups | Present on `Production` (commit `dd0be3e`) |
| `migrations/20261125_password_auth.sql` | Adds password hash storage and case-insensitive email index | Present on `Production`; user reports manual application to Supabase |

Email/password is a separately requested extension: the Master PRD and MVP brief specify Google sign-in. Google sign-in remains available, and frontend `main` now calls the email/password routes.

## Frontend integration on `main`

The frontend connects the static web screens to the backend through `src/js/api.js`, `src/js/integration.js`, and `src/js/admin-portal.js`. The admin sign-in, collector KYC review with ID/selfie evidence, dispute evidence and resolution controls, household dashboard, and latest collection/collector screens use existing backend routes. No new endpoint is required. This is web integration; it does not provide the PRD's native Android collector app.

The local backend's Supabase Session pooler passed a read-only `SELECT 1` check. The seven tables and columns used by the routes were present, and four material-rate rows were available. A deployed backend needs its own private `DATABASE_URL` and matching `GOOGLE_CLIENT_ID`; the deployed frontend needs its API base URL and an authorized Google JavaScript origin. OTP and media upload limitations remain documented in `API-GUIDE.md`.

## Repository layout

- Backend: [Ogundana-ST/capstone-project-backend](https://github.com/Ogundana-ST/capstone-project-backend), branch `Production`.
- Frontend: [ateelabo/recycleconnect](https://github.com/ateelabo/recycleconnect).
- API contract: this standalone repository; it contains specifications and integration documentation, not application code.

The local backend checkout used for the update is `backend-git/`; the successful push advanced `Production` to commit `dd0be3e`. The local `.env` is ignored by Git and holds the private Session pooler URL. Never commit or publish it.

Update this contract alongside future backend changes, and recheck each route's implementation status against the target branch.
