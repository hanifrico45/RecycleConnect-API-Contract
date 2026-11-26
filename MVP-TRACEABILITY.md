# MVP Traceability

This document uses the November 2026 MVP Brief and Master PRD as the scope source and compares them with the inspected backend `Production` and frontend `main` branches. It also records the separately requested email/password and profile routes pushed to backend `Production` in commit `dd0be3e`.

Status meanings: **Implemented** means the route exists on the inspected remote branch; **Partial** means only some acceptance criteria are implemented or an implementation constraint remains; **Missing** means no corresponding implementation was found in the inspected repositories. A backend route does not mean a frontend flow is connected.

| Requirement | Backend evidence | Frontend evidence | Status and notes |
| --- | --- | --- | --- |
| FR-01 household/estate sign-in and onboarding | Google sign-in; email/password extension; `PATCH /users/onboarding`; estate interval is forced to weekly | Signup/login and onboarding screens call the backend routes through the shared client | **Partial.** Integration exists. Local Google Client ID values match, but a live Google sign-in still needs the browser origin authorized and the backend restarted after environment changes. Household phone is required by the current signup form although the MVP says optional; PRD says estate Saturday 9 AM, so timezone/schedule behavior still needs review. |
| FR-01 collector OTP and KYC | OTP verify, KYC submit, admin approve/reject, approved-collector guard | Collector web workspace calls the backend routes; no native Android app in this repository | **Partial.** OTP is fixed demo value; API accepts document URLs but has no media upload or live-camera enforcement. |
| FR-02 live rates and household dashboard | `GET /rates`, `GET /pickups/dashboard` | Dashboard displays backend rates, wallet, pickup, and history through the shared client | **Partial.** API wiring exists; local Supabase schema and four material-rate rows were confirmed. Recheck deployed database setup before release. |
| FR-03 recurring, skip, pause, one-off pickup | Onboarding creates recurring pickup; skip, pause toggle, one-off route implemented | Household screens call these endpoints | **Partial.** One-off sends a photo reference; no upload endpoint. Confirm scheduled dates and address behavior against the MVP. |
| FR-04 verified dispatch, badge and map pins | Admin KYC decision; approved map pins; route status updates; notification text; dashboard badge | Collector and admin web screens use the relevant endpoints | **Partial.** Map response includes materials despite the brief's pins-only rule. Notification/dashboard do not return collector photo. |
| FR-05 weighing and conditional photo | Weigh route validates positive weights and requires scale photo URL when household is absent | Collector web workspace submits weights and the conditional photo reference | **Partial.** Server cannot enforce camera-only capture, photo compression, or image authenticity. |
| FR-06 instant wallet credit and confirmation/dispute | Weigh transaction updates wallet and ledger; confirm endpoint supports thumbs up/down | Household dashboard displays balances/history and sends confirmation or dispute | **Partial.** No explicit points balance or redemption endpoint; payout rounding should be agreed. |
| FR-07 admin KYC, rates, disputes, daily totals | Admin endpoints exist for each operation | Admin web workspace reads totals, edits rates, reviews collector ID/selfie evidence, and resolves disputes | **Partial.** The web portal supports manual evidence review; it does not automate identity matching. |
| Email/password signup, login, and profile update | Routes pushed to backend `Production` in commit `dd0be3e` | Frontend calls these routes through the shared client | **Integrated extension.** Email/password extends the PRD's Google-sign-in flow and was requested separately. |
| MMP payment-provider payouts | None found | None found | **Out of MVP** per PRD. |
| MMP WhatsApp bot and later roadmap items | None found | None found | **Out of MVP**; not included in this contract. |

## PRD endpoint list versus backend

The PRD says “17 Endpoints,” but its combined “GET /admin/overview & /disputes” row names two separate paths. Enumerating those paths gives 18 operations; the backend also has `PATCH /admin/disputes/{id}/resolve`, required by FR-07 but omitted from that table. The remote backend exposes 19 versioned operations. The backend `Production` branch now includes three additional operations (`POST /auth/signup`, `POST /auth/login`, and `PATCH /users/profile`) for 22 total. See `API-GUIDE.md` and `BACKEND-PARITY.md` for their status.

## Technology and ownership differences

- The early README DOCX describes a web application and Tailwind CSS CLI, with several features marked “to be confirmed.” The frontend repository matches static HTML/CSS/JavaScript with Tailwind CLI; it is not React Native/Expo.
- The PRD suggests React/Vite or Next.js for web, and Expo or Flutter for the collector app. Those are alternatives in the PRD, not the technology used by the existing static frontend. No framework migration is proposed here.
- The backend repository uses Express and PostgreSQL `pg`, matching one of the PRD's backend alternatives. It does not use Supabase Auth or the Supabase client. A PostgreSQL deployment may still be hosted by Supabase, but the repository only reads `DATABASE_URL`.
- Frontend `main` now includes household/estate, collector, and admin web integration. The PRD's separate native Android collector app is not present in this frontend repository.

