# 28 · Testing and quality (written 29 Oct 2026)

This is the project's TESTING page: what is checked, how to run it, how to add a test, and what the pipeline does.
Run everything in the code folder `vbw-crm`.

> **Update 30 Oct 2026:** there are now 127 API integration tests, 28 real-browser tests (`npm run test:e2e`) and 70 unit tests. See [31](31-launch-gaps-browser-tests-monitoring.md) for the browser, accessibility, security-policy and monitoring checks.

## 1 · The five checks

| Check | Command | What it needs |
|---|---|---|
| Lint (style and bug warnings) | `npm run lint` | nothing |
| Types | `npm run typecheck` (also the API's test folder; the Android app has its own: `cd apps/mobile && npx tsc --noEmit`) | nothing |
| Unit tests (60) | `npm test` | nothing |
| Build | `npm run build` | nothing |
| **Integration tests (108)** | `npm run test:integration` | database, Redis, API **and worker** running |
| Lint + types + unit tests + build in one go | `npm run check` (took 61 seconds on this PC) | nothing |

Rule (from the blueprint's definition of done): none of these may fail or warn before a change is committed.
`npm run lint` uses `--max-warnings 0`, so a warning counts as a failure.

## 2 · Running the integration tests

They talk to the real API with the development login (`AUTH_DEV_BYPASS=true`), the real database and Redis, like the app does.

```bash
npm run db:up          # database, Redis, file storage (Docker Desktop must be running)
npm run migrate
npm run dev:api        # terminal 1
npm run dev:worker     # terminal 2: webhooks are sent by the worker
npm run test:integration
```

- Every test creates its **own fresh accounts**, so they never depend on seeded data, can be repeated, and do not clean up
  after themselves (the local database fills with `it-…` test accounts; wipe it with `npm run db:down` and up again if you like).
- The tests clear the API's request counters (sign-up is limited to 5 an hour per address) through Redis; only the local test setup does this.
- API address and database settings can be overridden: `API_URL`, `DATABASE_ADMIN_URL`, `REDIS_URL`.
- If the API is not running, the tests stop at once with a clear message instead of failing 80 times.
- Production is protected: the development login refuses to start with `NODE_ENV=production`, so these tests cannot run against it.

## 3 · What the integration tests cover

| File | Covers |
|---|---|
| `flows.test.ts` | Sign up (100-day trial, 20 GB, terms), add / edit / find / move a customer, invites (wrong person, unverified email, used invite, third person), tasks done by the other person, invalid input, owner-only actions, one browser per person, plan ended (view-only) |
| `isolation.test.ts` | One business cannot read, edit, delete, merge, note, restore or search another's customers, appointments, tasks, follow-ups, properties, deals or offers; a smuggled business id is ignored; the database login cannot skip row security; every table with business data has it on; a cross-business reference is refused inside the database |
| `developer-api.test.ts` | Keys (look, hash-only storage, owner-only, revoke), caller lookup in any phone format, one lead per phone, booking with taken-slot and past-time refusal, call logging with "API: key name" in the history, read-only keys, keys stay inside their business, plan ended, 10-key limit, unsafe webhook addresses, secret shown once, **signed webhooks delivered** for an API event, an app event and a test event |
| `booking.test.ts` | Public page, enquiries tagged by source with a call-back task, booking requests, known phone reused, optional "send me offers" tick, spam (hidden field, too fast, no consent, bad phone, past time, unknown service), 7th enquiry in 10 minutes refused, paused booking, address clash |
| `leads.test.ts` | Lead addresses (made by the owner, shown once, hash only), every lead format, repeats ignored, bad and switched-off addresses, plan ended, isolation, log limit, test lead, any website may post, real estate fields; Hot / Warm / Cold cases, ties, Home card ([30](30-lead-capture-and-scoring.md)) |
| `industries.test.ts` | Each of the five industries' words and fields, the old `clinic` key refused, retail purchases and totals, dashboard cards, birthday in the pop-up, offers (who is counted, who is left out, STOP line, each send noted, consent withdrawn means no more sending), real estate matching (type, BHK, budget, location, available only), rented deal with commission and 11-month renewal, property share noted, Excel export |

Not covered by automated tests yet: file upload and download, payment links and Razorpay, Firebase login with real keys, email sending,
backups and restore (checked by hand, docs 21 and 23), the web screens (checked by hand in the browser at phone and desktop width),
and the Android app (type-checked only).

## 4 · Adding a test

1. Put it in the file for its area, or a new `apps/api/test/integration/<area>.test.ts`.
2. Start with `beforeAll(requireApi)` and `afterAll(closeDb)`; use `createAccount(industry)` (and `addMember(owner)`, `addContact(...)`) from `helpers.ts`.
3. Call the API with `call(client, 'POST', '/path', body)`; the answer is `{ status, body }`. For the Developer API use `callWithKey(key, ...)`.
4. Test the refusal as well as the success (wrong person, bad input, plan ended). Those are the cases that protect customers.
5. Run `npm run lint` and `npm run typecheck`, then the test twice in a row to be sure it does not depend on leftovers.

A good sign the tests work: the first run caught a real bug (a rented flat was still matched to clients) before any customer saw it.

## 5 · The pipeline (GitHub Actions, `.github/workflows/ci.yml`)

On every push to `main` and every pull request: install, **lint**, **type-check**, unit tests, build, apply the migrations on a clean
PostgreSQL, a database-level row-security check, then start the API and worker and run the **108 integration tests**; the Android app is type-checked in a second job.
It has not run on GitHub yet (nothing is pushed); the steps were checked locally, so expect small fixes on the first push.

## 6 · Code style

- One linter for everything: `eslint.config.mjs` (JavaScript and TypeScript recommended rules, React hooks rules, Next.js rules for the web app).
- A name starting with `_`, or a value taken out with `{ unwanted, ...rest }`, is allowed to stay unused.
- Silence a rule only on the one line, with a reason next to it (see the QR code image in `BookingTab.tsx`).
- Keep new comments to the "why", in plain words, like the existing ones.
- Business rules live in services; input is checked with zod on the server; every database change is a numbered migration.
