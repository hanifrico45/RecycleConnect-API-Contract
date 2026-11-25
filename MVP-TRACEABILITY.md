# MVP Traceability

This document uses the November 2026 MVP Brief and Master PRD as the scope source and compares them with the inspected backend `Production` and frontend `main` branches. It also records the separately requested email/password and profile routes pushed to backend `Production` in commit `dd0be3e`.

Status meanings: **Implemented** means the route exists on the inspected remote branch; **Partial** means only some acceptance criteria are implemented or an implementation constraint remains; **Missing** means no corresponding implementation was found in the inspected repositories. A backend route does not mean a frontend flow is connected.

| Requirement | Backend evidence | Frontend evidence | Status and notes |
| --- | --- | --- | --- |
| FR-01 household/estate sign-in and onboarding | Google sign-in; `PATCH /users/onboarding`; estate interval is forced to weekly | Signup/login forms are placeholders; onboarding stores values in local storage | **Partial.** Google-only matches source scope. Frontend is not wired. PRD says estate Saturday 9 AM; implementation schedule is code-defined and needs a timezone check. |
| FR-01 collector OTP and KYC | OTP verify, KYC submit, admin approve/reject, approved-collector guard | No collector mobile app in the frontend repository | **Partial.** OTP is fixed demo value; backend accepts URL fields but has no media upload. No admin UI/client. |
| FR-02 live rates and household dashboard | `GET /rates`, `GET /pickups/dashboard` | Dashboard uses local storage; reviewed frontend has no API calls | **Partial.** Backend route data exists; client integration is missing. |
| FR-03 recurring, skip, pause, one-off pickup | Onboarding creates recurring pickup; skip, pause toggle, one-off route implemented | Household pages are local-storage interactions | **Partial.** No live client integration. One-off requires a photo URL; no upload endpoint. |
| FR-04 verified dispatch, badge and map pins | Admin KYC decision; approved map pins; route status updates; notification text; dashboard badge | No collector map/client or verified-collector details wired to the household UI | **Partial.** Map currently returns materials despite the brief's privacy rule. Notification/dashboard do not return collector photo. |
| FR-05 weighing and conditional photo | Weigh route validates positive weights and requires scale photo URL when household is absent | Collector client absent | **Partial.** Server cannot enforce camera-only capture, photo compression, or image authenticity. |
| FR-06 instant wallet credit and confirmation/dispute | Weigh transaction updates wallet and ledger; confirm endpoint supports thumbs up/down | Dashboard displays local wallet data; no API integration | **Partial.** No explicit points balance; payout rounding should be agreed. |
| FR-07 admin KYC, rates, disputes, daily totals | Admin endpoints exist for each operation | No admin portal in frontend repository | **Partial.** API present, user interface absent. |
| Email/password signup, login, and profile update | Routes pushed to backend `Production` in commit `dd0be3e` | Frontend integration remains pending | **Backend routes are available; frontend wiring remains.** Email/password extends the PRD's Google-only signup flow and was requested separately. |
| MMP payment-provider payouts | None found | None found | **Out of MVP** per PRD. |
| MMP WhatsApp bot and later roadmap items | None found | None found | **Out of MVP**; not included in this contract. |

## PRD endpoint list versus backend

The PRD says “17 Endpoints,” but its combined “GET /admin/overview & /disputes” row names two separate paths. Enumerating those paths gives 18 operations; the backend also has `PATCH /admin/disputes/{id}/resolve`, required by FR-07 but omitted from that table. The remote backend exposes 19 versioned operations. The backend `Production` branch now includes three additional operations (`POST /auth/signup`, `POST /auth/login`, and `PATCH /users/profile`) for 22 total. See OpenAPI and `BACKEND-PARITY.md` for their status.

## Technology and ownership differences

- The early README DOCX describes a web application and Tailwind CSS CLI, with several features marked “to be confirmed.” The frontend repository matches static HTML/CSS/JavaScript with Tailwind CLI; it is not React Native/Expo.
- The PRD suggests React/Vite or Next.js for web, and Expo or Flutter for the collector app. Those are alternatives in the PRD, not the technology used by the existing static frontend. No framework migration is proposed here.
- The backend repository uses Express and PostgreSQL `pg`, matching one of the PRD's backend alternatives. It does not use Supabase Auth or the Supabase client. A PostgreSQL deployment may still be hosted by Supabase, but the repository only reads `DATABASE_URL`.
- The frontend repository includes household/estate page markup but no API integration. Admin and collector client responsibilities remain outside that repository's current contents.


