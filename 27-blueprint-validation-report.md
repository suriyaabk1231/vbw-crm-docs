# 27 · Blueprint validation report (29 Oct 2026)

Checked VBW CRM against the attached *Vibe Blueprint — Project Documentation* (PRD, MVP scope, acceptance criteria,
validation checklist, definition of done, 8-doc structure). Everything below was run today on the local copy; nothing was
changed in the product while checking.

## Update, same day: gaps 1 and 2 closed

Decision: the blueprint is the **method for VBW CRM** (not a second product). The first two next steps are done:

- **Integration tests are now in the repo** (`apps/api/test/integration/`, 81 tests in 5 files): sign up, records, invites, tasks, invalid input,
  permissions, plan ended, isolation between businesses (including the database itself), Developer API with signed webhooks, booking page with
  spam checks and rate limit, and the five industries with offers and property matching. Each test makes fresh accounts, so a run needs no
  seeded data and can be repeated; two runs in a row both passed 81 of 81. How to run and extend them: [28 — Testing and quality](28-testing-and-quality.md).
- **Lint is set up** (one `eslint.config.mjs` for the whole repo, zero errors and zero warnings; `npm run lint`). I checked it catches real
  problems (conditional hooks, unused variables, missing hook dependencies). The first run found only 22 findings in 123 files, all configuration
  scope or intentional patterns; no code bugs.
- **The new tests found a real bug on their first run:** a client was still "matched" to a property that was already rented or sold (and the
  "send to all matching clients" offer would have included them). Fixed in `sales.module.ts`: only available properties match.
- **Correction to section 5, gap 1:** GitHub Actions already existed (`.github/workflows/ci.yml`: tests, build, migrations, and a database-level
  row-security check). What it lacked was lint, type-checks and the API-level tests. They are added now. I checked the YAML parses and booted
  the API and worker with exactly the minimal settings the workflow uses and ran the 81 tests against them, but **the workflow itself has not
  run on GitHub yet**, so expect to fix small things on the first push.

## 0 · One conflict to settle first

The blueprint describes **a different product**: "Vibe Blueprint", a collaborative workspace where product builders
manage projects and tasks (web only, Next.js + Express + **Clerk** + **Vercel**, burgundy / ivory / Inter design).
VBW CRM is a CRM for Indian businesses (NestJS + **Firebase** + own server, teal design, ₹599 plan, WhatsApp,
five industries). The blueprint's own rule for conflicts is to state them and ask, not guess.

So this report uses the blueprint as a **method**: its MVP acceptance criteria, validation checklist, definition of
done and 8-document structure, mapped onto VBW CRM. It does **not** change VBW CRM's stack, design or features to match
the other product. If you meant to build Vibe Blueprint as a second product, that is a separate project (see section 7).

## 1 · Validation checklist (blueprint section 4)

| Check | Result | Evidence |
|---|---|---|
| Tests pass | ✅ | 32 of 32 (core 13, API 19) |
| Types check | ✅ | API, web, mobile and core all compile with no errors |
| Build succeeds | ✅ | Full production build, 23 pages |
| Lint passes | ✅ (set up later the same day) | First run: `next lint` asked to create a config because the repo had no ESLint. Now `npm run lint`: 0 errors, 0 warnings |
| Critical flows verified | ✅ | 39 of 39 checks, section 2 |

## 2 · MVP acceptance criteria (blueprint section 1)

**Criterion 1: critical flows work** (sign up, create a record, invite a member, complete a task): ✅ all four pass.

| Flow | Checked |
|---|---|
| Sign up | New account; same login twice refused; industry chosen; setup finished; trial set to 100 days |
| Create a record (patient) | Add, edit, open, search |
| Invite a member | Owner invites; wrong person refused; member joins and sees the same patients; used invite refused; a third person refused (account is for 2) |
| Complete a task | Owner adds a task; member completes it; shows under Done; reopen works |

**Criterion 2: edge cases handled** (invalid input, permissions, error states): ✅

| Area | Checked |
|---|---|
| Invalid input | Empty name, bad email, 500-character name, malformed id (400); unknown id (404); same phone twice (409); empty task title; broken JSON; script text stored as plain text |
| Permissions | No login (401); no registered device (401); member refused for export, invite, business settings, API keys (403); another business gets "not found" for read, edit and task completion (404) |
| Error states | Plan ended: can still read and export, cannot add (402). Server or internet down: shows "Cannot reach VBW CRM" with Try again, not the login page (fixed 28 Oct) |
| Earlier suites rerun today | Business isolation (all 404 across businesses, database role cannot bypass row security, 20 tables hold business data and all have it on except one the app cannot touch), Developer API, booking page, email-verified invites |

Two of my first-run expectations were wrong, not the app: a used invite answers 404 "invalid or expired" (the same as an
unknown one, so tokens cannot be guessed), and a missing device answers 401 `DEVICE_REQUIRED`. I corrected the expectations
and recorded both here. Some older test scripts also failed at first because they held stale browser ids (one browser per
person logs the older one out); that was the test setup, now refreshed.

**Criterion 3: build and tests pass:** ✅ except lint (above).

## 3 · MVP scope mapped to VBW CRM (blueprint section 1)

| Blueprint MVP item | VBW CRM equivalent | Status |
|---|---|---|
| Authentication (email + OAuth) | Firebase: email and password, Google, branded reset, verified email to join a team | ✅ built; **only tested with the local dev login, not real Firebase keys yet** |
| Create, edit, organise projects | Customers with fields, tags, stages, pipeline | ✅ |
| Basic task management (list view) | Tasks (today, overdue, upcoming, done) | ✅ |
| Team collaboration (comments, mentions) | Notes and history on each customer, 2 people per account | 🟡 notes yes; no @mentions or comment threads (not needed for 2 people) |
| Real-time notifications | Pop-up on opening the app and every 2 minutes while open; booking-page enquiries; birthdays | 🟡 near-real-time by checking, not push |
| Responsive web | Checked at 375 px on landing, setup, properties, client page, offers; Settings and tabs checked earlier | ✅ |

Blueprint "later" items: Kanban (pipeline ✅), calendar (✅), integrations (Developer API and webhooks ✅), advanced
permissions (owner and member only), analytics (basic dashboard cards), mobile apps (Android app built, behind the web app).

Beyond the blueprint's MVP, VBW CRM already has: WhatsApp one-tap messages, renewals and fees, payment links (off until
Razorpay), file uploads (20 GB), online booking page, offers, five industries, Developer API, backups and one-business restore.

## 4 · Documentation coverage (blueprint's 8-doc set)

| Blueprint doc | In VBW CRM | Gap |
|---|---|---|
| PRD.md | 01, 02, 03, 08, 26 | **No written MVP "in / out" list and no acceptance criteria.** Section 2 above is the first one |
| ARCHITECTURE.md | 04, 05, 06 | none |
| DESIGN_SYSTEM.md | Colour tokens in `apps/web/src/app/tokens.css`; design prompts 18, 19 | **No single design-system doc** (colours, type, spacing, states, accessibility) |
| AGENTS.md | Handover (20) only | **Missing**: how a coding agent or new partner works on the code (inspect, plan, implement, test, review, document) |
| DATABASE.md | Migrations 001 to 010; row-security rules in 23 | **Missing**: one page with the tables, relations and rules |
| SECURITY.md | 07 and 23 | none (could be merged into one page) |
| CODE_STYLE.md | none | **Missing** (no linter either) |
| TESTING.md | none | **Missing** |

## 5 · Gaps found, most important first

| # | Gap | Why it matters | Fix |
|---|---|---|---|
| 1 | ✅ **Done: integration tests are now in the repo** (see the update at the top). Before: **integration tests were not in the repo.** The 39-check acceptance script and the isolation, Developer API, booking and invite scripts live in a temporary folder. The repo has only 32 unit tests (core 13, API 19 on access log, retention and email helpers) | The most important promises (one business cannot see another's data; members cannot do owner actions) are checked by hand, not by every code change; a partner or CI cannot rerun them | Move them into the repo (`infra/tests/`), run them in the deploy script and, later, in GitHub Actions |
| 2 | ✅ **Done: lint** (see the update at the top) | Blueprint checklist item | `npm run lint`; also runs in CI and in the build |
| 3 | **Code is only on this computer.** Neither repo has a GitHub remote; the last commits are not pushed | One disk failure loses everything | Push both repos (your step) |
| 4 | **23 package vulnerabilities** (9 high, 14 moderate, 0 critical) in production dependencies | Needs major version upgrades; high ones should be reviewed before launch | Upgrade in one planned session, then rerun tests |
| 5 | **Real Firebase login, real email and Razorpay were never tested with real keys** | The first real sign-up could fail on something the dev login hides | Test in the cloud with real keys before the pilot (doc 22) |
| 6 | **No retry in the web app's calls** (blueprint: "retry safely with exponential backoff") | A brief network blip shows an error instead of retrying quietly. Webhooks do retry | Retry safe reads (GET) twice with a short wait |
| 7 | Missing docs: AGENTS, DATABASE, CODE_STYLE, TESTING, DESIGN_SYSTEM, MVP list | Blueprint's "docs updated" rule; slows a new partner | Write them (section 6) |
| 8 | Still open from earlier: backup bucket lock (G5), second backup copy (G6), no automatic STOP handling for offers, Android app lacks Properties / Purchases / Offers, no admin panel for your own team | Known, each tracked in docs 23, 24, 26 | See next steps |

## 6 · Definition of done (blueprint section 4), applied to the last stretch of work

| Item | Status |
|---|---|
| Requirements satisfied | ✅ Developer API, five industries, offers, outage fix |
| Changes reviewed | 🟡 No second person has reviewed the code; no pull request process yet |
| Docs updated | ✅ docs 25, 26, this report; README and decisions log |

## 7 · Next steps

**Before the pilot (in this order):**
1. ~~Move the test scripts into the repo~~ ✅ done (`npm run test:integration`).
2. ~~Add lint~~ ✅ done (`npm run lint`; everything together: `npm run check`).
3. **Push both repos to GitHub** (you).
4. **Write the missing docs**: MVP list with acceptance criteria, AGENTS, DATABASE, TESTING, CODE_STYLE, DESIGN_SYSTEM (gap 7). About one day, mostly from what already exists.
5. **Upgrade vulnerable packages** and rerun everything (gap 4).
6. **Cloud setup** from doc 22, then test real Firebase, email and backups on the live server (gap 5). Needs your domain extension.
7. **Admin panel** for your own team (design agreed on 28 Oct): before the pilot accounts arrive.
8. Retry for web reads (gap 6), lock the backup bucket (G5), lawyer review of terms, privacy and offers consent.

**Your decision:** is the blueprint meant as the **method for VBW CRM** (as used here), or do you also want
**Vibe Blueprint built as its own product**? If the second, tell me and I will start from its PRD and ask about the
parts the PDF leaves open (SSO choice, real-time method, who the admin role is, where Vercel and Clerk fit your budget).
