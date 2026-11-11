# 35 — VBW CRM: complete system overview (tools, front end, back end, APIs, status)

*Written 30 Oct 2026 from the code in `vbw-crm` (not from memory). Counts of routes, tables, jobs, pages and tests were read from the code and by running the test suites that day. "Built" means built and tested on the local copy. "Not done" means it needs servers, accounts or a decision.*

---

## 1. What it is

**VBW CRM** is a client-relationship system for Indian growing businesses, one setup per industry: **Hospital, Gym & Fitness, Education Centre, Retail Shop, Real Estate**. One **account** = 2 people (Owner + Member), ₹599 a month + GST, 20 GB of files shared, 100-day trial. Web app first, Android app next.

---

## 2. Every tool and technology we use

### 2.1 Front end — web (`apps/web`)

| Tool | Version | Used for |
|---|---|---|
| Next.js | 15.5.28 | The website and the app screens (server-rendered on request, because of the security nonce) |
| React | 19.0.0 | Screen building |
| Tailwind CSS (+ own CSS files `app.css`, `tokens.css`, `glass.css`, `pulse.css`) | — | Styling, light and dark mode |
| Firebase (web SDK) | ^11.1.0 | Login in the browser (email + password, Google) |
| lucide-react | 1.54.0 | Icons |
| marked | 15.0.12 | Shows the Terms / Privacy texts (Markdown) |
| qrcode | 1.5.4 | QR code for the online booking page |
| Playwright + axe-core | — | Real-browser tests and accessibility scans |
| Content Security Policy | — | A per-request nonce set in `src/middleware.ts` / `src/lib/csp.ts` |

### 2.2 Front end — Android (`apps/mobile`)

| Tool | Version | Used for |
|---|---|---|
| React Native | 0.79.6 | The Android app |
| Expo (SDK 53) + expo-router | ~53.0.9 / ~5.1.11 | Build, navigation |
| Firebase (JS SDK) | ^11.1.0 | Login |
| expo-notifications | ~0.31.2 | Push notifications |
| expo-image-picker / expo-image-manipulator | ~16.1.4 / ~13.1.7 | Camera photo, compress to ~1600 px |
| expo-document-picker | ~13.1.6 | Pick PDFs, scans, Excel for a client (new, 30 Oct) |
| expo-location | ~18.1.5 | "Check in" at a visit |
| expo-secure-store, async-storage | — | Keeping the login safely on the phone |

### 2.3 Back end (`apps/api`)

| Tool | Version | Used for |
|---|---|---|
| Node.js | 20 or newer (CI uses 22) | Runs everything |
| NestJS | ^10.4.15 | The API (controllers, services, guards) |
| TypeScript | — | All code is typed |
| PostgreSQL | 16 (in CI) | The database, with **row-level security** (each business sees only its own rows) |
| `pg` | ^8.13.1 | Database driver (two logins: app login with security rules, owner login for jobs and sign-up) |
| Redis (ioredis ^5.4.2) | 7 | Rate limits, cache, job queue (capped at 512 MB) |
| BullMQ | ^5.34.0 | Background jobs (reminders, retention, backups, webhooks, automations) |
| Zod | ^3.24.1 | Checks every request body |
| firebase-admin | ^12.7.0 | Checks login tokens, sends push |
| @aws-sdk/client-s3 + presigner | ^3.716.0 | Files in Cloudflare R2 (S3-compatible) |
| ExcelJS | ^4.4.0 | Excel import and export |
| helmet | ^8.0.0 | Security headers |
| Brevo (HTTP API) | — | Sends email (reset, trial, alerts). Needs `BREVO_API_KEY` |
| Razorpay | — | Online payments (code ready, **off** until keys are added) |

### 2.4 Shared code (`packages/core`)

Plain TypeScript used by API, web and Android: the price and storage limits (`constants.ts`), the five industry templates (`templates.ts`), lead scoring and lead-reading (`leads.ts`), automation rules (`automations.ts`), WhatsApp link building and formatting helpers (`helpers.ts`).

### 2.5 Infrastructure and operations (`infra`, `.github`)

| Tool | Used for |
|---|---|
| Docker + Docker Compose | Runs everything the same on a PC and a server (`docker-compose.yml`, `docker-compose.dev.yml`) |
| Caddy | Web server, HTTPS in front of the app (`Caddyfile`) |
| PostgreSQL config + **pgBackRest** | Encrypted continuous backups to R2 (`infra/postgres`) |
| Standby scripts | A live copy of the database on a second server (`infra/standby`: `setup-replica.sh`, `promote.sh`) |
| WireGuard | Private link between the two servers (`infra/wireguard`) |
| Shell scripts | `deploy.sh`, `rollback.sh`, `backup.sh`, `harden-server.sh`, `restore-business.sh`, `restore-test.sh` |
| **k6** | Load test script `infra/load-test/k6.js` (never run on a real server yet) |
| GitHub Actions | `ci.yml`: lint, type-check, tests, build, browser tests, with Postgres 16, Redis 7 and an S3 stand-in. **Never run on GitHub yet** |
| adobe/s3mock | Stands in for R2 on the local PC and in CI |

### 2.6 Outside services (plan; most accounts not created yet)

| Service | Role | Status |
|---|---|---|
| Cloudflare (DNS, HTTPS, firewall) | In front of the site | Free; account not set up |
| Cloudflare R2 | Customer files, copy of files, database backups, logs | Code works against a stand-in; **no real R2 test** |
| Firebase Authentication | Login (email, Google) | **Dev login bypass used locally**; real test pending |
| Firebase Cloud Messaging | Push to the Android app | Code written; not tested for real |
| Brevo | Email sending | Code written; no key yet |
| Razorpay | Online payments (₹599 plan) | Code ready; plan and keys not created |
| DigitalOcean (main) + Contabo (standby) | Servers | **Not bought** (docs 22 and 34) |

---

## 3. How the parts fit together

```
 Browser (Next.js)        Android (Expo)        Other software (Developer API key)
        \                      |                      /
         \                     |                     /
          +------------>  Caddy (HTTPS)  <-----------+
                               |
                       API (NestJS, /api/v1)  ---- Firebase (checks login tokens)
                       |        |        \
                 PostgreSQL   Redis     R2 (files via short-lived upload/download links)
                       |        |
                    pgBackRest  Worker (BullMQ jobs) ---> Brevo (email), Firebase (push), customer webhooks
                       |
                    R2 backups  +  standby server (live copy)
```

- **Every request** is checked for login, for the person's role (Owner or Member), for the plan (view-only after the trial or lapse), and is limited to its own business by the database.
- **Files never go through the API.** The API hands out a short-lived address; the browser or phone sends the file straight to R2, then tells the API to confirm and count the size.
- **The worker** (`apps/api/src/worker.ts`) is a separate process that runs scheduled jobs.

---

## 4. What is built, end to end

### 4.1 Account and team

- Sign up with a 100-day trial that ends exactly at sign-up + 100 days; pick one of five industries; short setup.
- Login with Firebase (email/password, Google); password reset with branded email.
- Two people per account; invite by link; one phone and one browser per person; a new login signs the older one out.
- Owner can remove a person or a device, delete the account (7-day grace), see an activity log, export everything to Excel.
- Plan states: trialing, active, past due, cancelled, expired. After the plan ends the account is **view-only** (export still works).
- Terms, Privacy, DPA and sub-processors pages (draft text); agreement version stored per person.

### 4.2 Clients (contacts)

- Add, edit, search, merge duplicates (phone/email), delete with 30-day restore, tags, custom fields per industry plus the owner's own fields.
- Import from Excel/CSV (preview, mapping, up to 5,000 rows) and export.
- History per client: calls (with outcome), notes, WhatsApp, stage moves, files, appointments, renewals, sales.
- **Files and reports (new):** attach several files (PDF, JPG, PNG, WEBP, XLSX, XLS, CSV, 25 MB each, 20 GB per account) on web and Android, also while adding the client; list with size and date; download; delete with restore for 30 days.

### 4.3 Pipeline, appointments, follow-ups, tasks

- Stages per industry, editable; one-tap moves; Won / Lost.
- Appointments (calendar, status booked/done/no-show/cancelled, online requests to confirm), renewals and follow-ups (recurring, due soon/overdue), tasks (assign, due date, complete, reopen, reminders).

### 4.4 Industry extras

- **Real estate:** Properties list, client-to-property matching, deals with commission, 11-month rent renewal, requirements captured from leads.
- **Retail:** purchases, birthday/anniversary, win-back of inactive customers.
- **Hospital / Gym / Education:** their own words and stages (patients, members, students; sessions; admissions; fees pending).

### 4.5 Leads, scoring and capture

- **Lead scoring:** Hot / Warm / Cold with every reason shown (`scoreLead`). Leads screen on web, badges on pipeline cards, Hot-leads card on Home.
- **Lead addresses:** private web addresses that other services post to (website forms, Google Ads lead form shape, Meta answers via Zapier/Make, IndiaMART-style, JustDial-style GET). A lead becomes a client with a call-back task; repeats are ignored; a log shows what arrived.
- **Online booking page** per business (`/b/<slug>`), QR code, requests become leads.

### 4.6 Offers and WhatsApp

- WhatsApp **click-to-send** (`wa.me`) with templates; one tap per person; offers go only to people who agreed. Not a two-way inbox.

### 4.7 Reports and automations (new, 30 Oct)

- **Reports:** by source (added, won, won %), people per stage, what each person did, 7 days to 1 year, owner-only Excel.
- **Automations (Settings, owner only, all off by default):** hot-lead call task, quiet-lead follow-up task (3–60 days), who gets new-lead tasks (owner or second person). Hourly job plus "Run now"; at most 20 tasks per rule per run.

### 4.8 Payments

- Online payment links and a monthly Razorpay subscription: **code ready, switched off** (`ONLINE_PAYMENTS`). Today customers pay ₹599 + GST by UPI or bank transfer and the owner activates the account with a command (`npm run activate`).

### 4.9 Developer API (for other software)

- Owner creates keys (read-only or read-and-change); keys are stored as a hash. Calls under `/ext/*` follow the same plan rules as people. Outgoing **signed webhooks** for new leads and enquiries, with retries. Published `openapi.json`.

### 4.10 Safety, privacy and operations (built)

- Row-level security; append-only audit log; access logs kept 365 days; rate limits; Content Security Policy; 1 MB body limit; secrets stored as hashes.
- Data retention: deleted clients and files erased after 30 days; lapsed accounts erased after 90 days with two warning emails; account deletion after a 7-day grace period.
- Backups (pgBackRest to R2, encrypted), second copy of every file, one-business restore, restore test script.
- Monitoring: `/health`, `/health/ready`, `/health/metrics` (token), push alerts to `ALERT_WEBHOOK_URL`, worker heartbeat to `HEARTBEAT_URL`.

---

## 5. The API: every route (129), grouped

All paths start with `/api/v1`. **Login** = a person's Firebase login. **Owner** = Owner only. **Public** = no login. **Key** = Developer API key.

### 5.1 Health (internal, for monitoring)

| Route | What |
|---|---|
| `GET /health` | Alive |
| `GET /health/ready` | Database, Redis and storage are reachable |
| `GET /health/metrics` | Counters (needs the `x-metrics-token` header) |

### 5.2 Account, team, settings

| Route | What |
|---|---|
| `POST /auth/signup`, `POST /auth/password-reset` | Create account, reset email |
| `POST /auth/devices`, `PATCH /auth/devices/current` | Register this browser/phone; save its push token |
| `GET /me`, `PATCH /me/preferences` | Who am I, plan, industry template; light/dark |
| `POST /setup/industry`, `POST /setup/complete` | Choose industry, finish setup |
| `PATCH /settings/business` | Business name, phone, city, own fields (Owner) |
| `GET /team`, `POST /team/invites`, `DELETE /team/invites/:id`, `POST /team/invites/accept` | Team and invites |
| `DELETE /team/members/:id`, `DELETE /team/devices/:id` | Remove a person or a device (Owner) |
| `POST/DELETE /account/deletion` | Ask for / cancel account deletion (Owner) |
| `GET /audit` | Activity log (Owner) |

### 5.3 Clients, pipeline, import and export

| Route | What |
|---|---|
| `GET/POST /contacts`, `GET/PATCH/DELETE /contacts/:id` | List, add, view, edit, delete |
| `GET /contacts/duplicates`, `POST /contacts/:id/merge` | Find and merge duplicates |
| `GET /recently-deleted`, `POST /contacts/:id/restore` | 30-day restore |
| `POST /contacts/:id/stage`, `POST /contacts/:id/activities` | Move stage; add a call/note/WhatsApp entry |
| `GET /pipeline`, `POST /pipeline/stages`, `PATCH/DELETE /pipeline/stages/:id` | Stages |
| `POST /import/preview`, `POST /import/commit`, `GET /export` | Excel import; full export (Owner) |

### 5.4 Appointments, renewals, tasks

| Route | What |
|---|---|
| `GET/POST /appointments`, `PATCH /appointments/:id`, `POST /appointments/:id/status` | Appointments |
| `GET/POST /renewals`, `PATCH /renewals/:id`, `POST /renewals/:id/status` | Renewals and follow-ups. Since 31 Oct a renewal can be created by length (`after`) or date, with reminder points (`remindDays`); renewing can upgrade (`nextAfter`) (doc 38) |
| `GET /due-soon`, `POST /renewals/:id/snooze` | The Due soon list; hide an item until a day (doc 38) |
| `GET/POST /tasks`, `PATCH /tasks/:id`, `POST /tasks/:id/complete`, `POST /tasks/:id/reopen` | Tasks |

### 5.5 Files

| Route | What |
|---|---|
| `POST /files/upload-url` | Check type, size and the 20 GB limit; get a short-lived upload address |
| `POST /files/:id/confirm` | Count the real size, copy to backup |
| `GET /files/:id/download-url`, `DELETE /files/:id` | Download address; delete |
| `POST /files/:id/restore` | Bring a deleted file back (Owner) |

### 5.6 Leads, booking and alerts

| Route | What |
|---|---|
| `GET /leads` | Hot / Warm / Cold list |
| `GET/POST /lead-inboxes`, `PATCH/DELETE /lead-inboxes/:id`, `GET /lead-inboxes/:id/log`, `POST /lead-inboxes/:id/test` | Manage lead addresses (Owner) |
| `GET/POST /public/leads/:token` | **Public**: other services post leads here; the secret in the address is the key |
| `GET/POST /public/booking/:slug` | **Public**: the booking page's details and requests |
| `GET/PATCH /booking/settings` | Booking page settings (Owner) |
| `GET /alerts`, `POST /alerts/seen` | New-enquiry pop-ups |

### 5.7 Messaging and offers

| Route | What |
|---|---|
| `GET/POST /message-templates`, `PATCH/DELETE /message-templates/:id` | WhatsApp message templates |
| `POST /whatsapp/link` | Build a `wa.me` link and record the action |
| `POST /offers/preview`, `GET/POST /offers`, `GET/DELETE /offers/:id`, `POST /offers/:id/sent/:contactId` | Offers to consenting clients |

### 5.8 Real estate and sales

| Route | What |
|---|---|
| `GET/POST /properties`, `GET/PATCH/DELETE /properties/:id`, `POST /properties/:id/shared` | Properties |
| `GET /contacts/:id/matching-properties` | Matching properties for a client |
| `POST /contacts/:id/sales`, `DELETE /sales/:id` | Purchases and deals |

### 5.9 Dashboard, reports, automations

| Route | What |
|---|---|
| `GET /dashboard` | Home cards and today's lists (cached 60 s) |
| `GET /reports`, `GET /reports/export` | Reports; Excel (Owner) |
| `GET/PUT /automations`, `POST /automations/run` | Automation settings and "Run now" (Owner) |

### 5.10 Payments (code ready, off)

| Route | What |
|---|---|
| `GET /billing`, `POST /billing/subscribe` | Plan status; start the monthly subscription |
| `POST/GET /payment-links` | Customer payment links |
| `POST /webhooks/razorpay` | **Inbound**: Razorpay tells us a payment or subscription changed (signature checked) |

### 5.11 Developer API (Key)

| Route | What |
|---|---|
| `GET /developer`, `POST/DELETE /developer/keys`, `PUT/DELETE /developer/webhook`, `POST /developer/webhook/rotate-secret`, `POST /developer/webhook/test` | Owner manages keys and the outgoing webhook |
| `GET /ext/openapi.json`, `GET /ext/me`, `GET /ext/business` | Description and identity |
| `GET/POST /ext/contacts`, `GET/PATCH /ext/contacts/:id`, `POST /ext/contacts/:id/calls`, `/notes`, `/sales` | Clients, calls, notes, sales |
| `GET /ext/availability`, `GET/POST /ext/appointments`, `PATCH /ext/appointments/:id` | Appointments |
| `GET /ext/followups`, `POST /ext/tasks`, `GET /ext/properties` | Follow-ups, tasks, properties |

**Outgoing (internal) calls:** signed webhooks `lead.created` and `enquiry.received` to the owner's address (BullMQ job `webhook-deliver`, with retries).

### 5.12 Background jobs (internal, run by the worker)

| Job | When | What |
|---|---|---|
| heartbeat | every 5 min | Pings the outside "is it alive" service |
| morning-summary | 08:00 | Morning push |
| appointment-reminders | every 5 min | Reminders |
| renewal-alerts | 09:00 | Follow-ups and renewals due |
| plan-reminders | 09:05 | One push per reminder point reached (the Due soon screen), naming the person |
| task-reminders | every minute | Task reminder push |
| automations | hourly, :10 | Hot-lead and quiet-lead tasks |
| mark-no-shows | 23:30 | Unmarked bookings become no-shows |
| trial-emails | 10:00 | Trial ending and manual-payment reminders |
| cleanup-pending-files | 03:00 | Remove unfinished uploads |
| purge-deleted-contacts / purge-deleted-files | 03:15 / 03:20 | Erase after 30 days |
| backup-files | 01:45 | Second copy of files |
| data-retention | 03:30 | Warn and erase lapsed accounts |
| archive-access-logs | 00:30 | Move access logs to storage |
| webhook-deliver | on demand | Customer webhooks |

---

## 6. The database (25 tables, 12 migrations)

`industry_templates`, `businesses`, `users`, `invites`, `devices`, `pipeline_stages`, `contacts`, `appointments`, `renewals`, `tasks`, `activities`, `message_templates`, `payment_links`, `files`, `audit_log`, `webhook_events`, `erased_businesses`, `api_keys`, `webhooks`, `properties`, `sales`, `campaigns`, `campaign_sends`, `lead_inboxes`, `lead_inbox_log`. (Migrations 001–011 in `apps/api/src/db/migrations`.) Business settings (own fields, automation settings) live as JSON in `businesses.settings`. Migration 012 adds `plan_length`, `plan_unit`, `remind_days`, `reminded_days` and `snoozed_until` to `renewals`.

---

## 7. Screens

**Web (28 pages):** landing page; login, sign-up, forgot/reset password, invite; Terms, Privacy, DPA, sub-processors; Home, Contacts and a client page, Calendar, Renewals, Due soon, Pipeline, Leads, Reports, Tasks, Offers (+ detail), Properties (+ detail), Import, Setup, Settings (Business, Team & devices, Stages, WhatsApp messages, Online booking, Lead sources, Automations, Developer API, Plan & billing, Data); public booking page `/b/<slug>`.

**Android (10 screens):** login, forgot password; tabs Home (with a bell and count for Due soon), Contacts, Calendar, Renewals, Tasks; Due soon; new client, client page (call, WhatsApp, photo, add files, check-in, stage, history); settings. **Not on Android yet:** Leads, Reports, Automations, Properties, Offers.

---

## 8. Testing (run on 30 Oct 2026)

| Suite | Count | Result |
|---|---|---|
| Core unit tests (`packages/core`) | 70 | passed |
| API unit tests | 29 | passed |
| API integration tests (real database) | 170 | passed |
| Browser tests (Chrome, real screens) | 39 | passed, incl. accessibility scans (light and dark) |
| Lint | — | clean (0 warnings) |

**Not tested:** on a real server; with real Firebase, R2, Brevo, Razorpay or Meta; the Android app on a phone; the load test; restore and failover drills; GitHub CI.

---

## 9. What is done and what is not

### Done (built and tested locally)
Everything in section 4, the API in section 5, the jobs, the safety and data-retention rules, the infra scripts (written), 308 automated tests passing, 35 documents.

### Needs your side (accounts, servers, decisions)
1. Push to GitHub and run CI once.
2. Create real Firebase, R2, Brevo accounts; buy servers (docs 21, 22, 34).
3. Test Google sign-in and real file uploads (use `CSP_REPORT_ONLY=true` first).
4. Load test (k6), restore drill, failover drill (doc 31).
5. Razorpay plan (₹599) and keys, when online payments start.
6. Final legal text; the pilot offer; whether pilots already promised ₹399 keep it.

### Not built (on purpose or deferred)
Two-way WhatsApp inbox (needs WhatsApp Business API); rule builder for automations; direct Meta Lead Ads, IndiaMART field-name confirmation, property-portal email reading; Indian-language screens; configurable reports, cohorts, campaign ROI; Leads/Reports/Automations on Android; Sentry error tracking; favicon; international auto-payments (needs a provider decision).

---

## 10. Where to read more

| Topic | Document |
|---|---|
| Step-by-step to-do list | 21 |
| Servers and monthly cost | 22, 06, 34 |
| Data safety and backups | 23 |
| Booking page and leads | 24, 30 |
| Developer API | 25 |
| Retail and real estate | 26 |
| Testing and quality | 28, 31 |
| Storage | 29 |
| Files, ₹599, 20 GB | 32 |
| Reports and automations | 33 |
| 100-account plan | 34 |
| Decisions log | 11 |
