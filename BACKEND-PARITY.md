# Backend Repository Parity

The API contract describes the linked GitHub backend branch `Production`. The email/password signup and login routes, profile route, environment template, and password-auth migration were pushed to that branch in commit `dd0be3e`.

## Backend additions on `Production`

| Route or file | Behavior | Status |
| --- | --- | --- |
| `POST /v1/auth/signup` | Email/password account creation | Present on `Production` (commit `dd0be3e`) |
| `POST /v1/auth/login` | Email or phone/password login | Present on `Production` (commit `dd0be3e`) |
| `PATCH /v1/users/profile` | Update profile fields without rebuilding recurring pickups | Present on `Production` (commit `dd0be3e`) |
| `migrations/20261125_password_auth.sql` | Adds password hash storage and case-insensitive email index | Present on `Production`; user reports manual application to Supabase |

Email/password is a separately requested extension: the Master PRD and MVP brief specify Google sign-in. Google sign-in remains available. The frontend still needs to integrate and review the routes.

## Repository layout

- Backend: [Ogundana-ST/capstone-project-backend](https://github.com/Ogundana-ST/capstone-project-backend), branch `Production`.
- Frontend: [ateelabo/recycleconnect](https://github.com/ateelabo/recycleconnect).
- API contract: this standalone repository; it contains specifications and integration documentation, not application code.

The local backend checkout used for the update is `backend-git/`; the successful push advanced `Production` to commit `dd0be3e`. The local `.env` is ignored by Git and still needs a private `DATABASE_URL` before database-backed routes can run.

Update this contract alongside future backend changes, and recheck each route's implementation status against the target branch.

