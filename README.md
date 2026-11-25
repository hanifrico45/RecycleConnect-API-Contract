# RecycleConnect API Contract

This is a standalone contract repository for the existing RecycleConnect frontend and backend. It contains API specifications and integration documentation only; it does not contain or replace either application. The frontend and backend remain in their own repositories.

## Connected project repositories

- Frontend: [ateelabo/recycleconnect](https://github.com/ateelabo/recycleconnect)
- Backend: [Ogundana-ST/capstone-project-backend](https://github.com/Ogundana-ST/capstone-project-backend)

The API definition is [`openapi.yaml`](openapi.yaml) (OpenAPI 3.1). Import it into Swagger Editor/UI or a compatible API client to browse the contract. [`API-GUIDE.md`](API-GUIDE.md) explains authentication, roles, errors, workflows, and implementation caveats. [`MVP-TRACEABILITY.md`](MVP-TRACEABILITY.md) distinguishes implemented, partial, missing, and frontend-only work. [`BACKEND-PARITY.md`](BACKEND-PARITY.md) records which contract routes are implemented by the linked backend.

## Current project responsibilities

- Frontend repository: static HTML, CSS, and browser JavaScript (Tailwind CSS CLI). Frontend `main` commit `b203df9` contains a shared API client and wired household, collector, and admin screens.
- Backend repository: Node.js, Express, PostgreSQL through `pg`, Google ID-token verification, and JWT bearer tokens.
- This repository: API agreement and MVP traceability only.

The repositories do not currently share generated code or runtime dependencies. The API uses JSON over HTTP. The backend's local development port defaults to `3000`; the PRD names `https://api.recycleconnect.ng/v1` as a target base URL, but that deployment has not been verified here.

## Contract status

The OpenAPI file describes the backend `Production` branch, including the email/password and profile-update routes pushed in commit `dd0be3e`. The frontend `main` branch now consumes the documented routes through its shared client (commit `b203df9`). The contract remains documentation; it does not contain either application's code.

Supabase hosts PostgreSQL for this project. The password-auth migration was applied manually to the project, but the local backend still needs its private `DATABASE_URL` configured. The backend repository has no checked-in database schema/setup script even though `package.json` declares `db:setup`. Database response row shapes are therefore not fully guaranteed by the source repository. See the unresolved questions in the guide before treating the contract as a production integration agreement.

## Updating this contract

1. Compare any proposed change with the MVP traceability document and both application repositories.
2. Mark each endpoint as implemented only after confirming it exists on the target backend branch.
3. Update `openapi.yaml` and the traceability notes in the same review.
4. Get frontend and backend review for changes to paths, payloads, roles, or state transitions.

This API contract is maintained in its own repository, separate from the frontend and backend repositories.


