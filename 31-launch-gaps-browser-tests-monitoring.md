# 31 · Launch gaps: browser tests, accessibility, security policy, monitoring (30 Oct 2026)

This page records the six "important gaps before launch" from the code review, what was done about each, what is still left, and how to run and set up everything. All numbers below were measured on this PC on 30 Oct 2026.

## 1 · Where each gap stands

| # | Gap | Status | What exists now |
|---|---|---|---|
| 1 | Real-browser tests: sign-up, login, setup, lead inbox, file upload, account expiry, owner/member | ✅ Done | `apps/web/e2e/`: 28 tests in a real Chrome. Run `npm run test:e2e`. Also in the CI pipeline. |
| 2 | Automated accessibility checks + keyboard/screen-reader QA | ✅ Automated done; ⏳ manual screen-reader pass is yours (checklist in section 5) | axe-core scans of 8 public pages and every signed-in page and settings tab, in light and dark; 3 keyboard tests. Two real problems found and fixed. |
| 3 | Production monitoring | ✅ Built; ⏳ needs your free accounts (section 4) | `/health/ready`, `/health/metrics`, push alerts, a "still alive" ping, job and payment-webhook tracking. |
| 4 | Content Security Policy | ✅ Done | One-time-code (nonce) policy on every page; tested against a simulated injection attack. |
| 5 | Staging load test, restore drill, backup verification, standby failover drill | ⏳ Needs real servers (section 6) | Scripts exist (`infra/load-test/k6.js`, `restore-test.sh`, `promote.sh`); nothing here can run them without servers. |
| 6 | Keep lead-capture changes together in a reviewed commit | ⏳ Waiting for your decision | The working tree still has the lead-capture work AND today's changes, uncommitted. See section 7. |

The recommended order from the review, and where it stands:

1. Fix vulnerabilities and the file-quota bypass: ✅ (section 2)
2. Bring up the local stack and run all integration tests: ✅ **127 of 127 pass** (108 old + 8 file + 3 import + 8 health)
3. Add desktop browser and accessibility tests: ✅
4. Legal/business launch inputs (Firebase, e-mail, Cloudflare/R2, payments, servers, Play Console): ⏳ yours
5. Deploy staging, test backup/restore and failover, then load test: ⏳ needs servers
6. Final legal pages and production monitoring before public launch: ⏳ monitoring is built, accounts and legal text are yours

## 2 · Fixes made

### File-quota bypass (security bug, fixed)

The 4 GB account limit could be exceeded two ways:

- **Many upload links at once.** The limit was checked only against files already confirmed, so one account could request dozens of links before confirming any of them.
- **A wrong size announced.** A link was requested for "1 byte" and then a 25 MB file was sent. The real size was never checked against the 4 GB limit.

Now: links handed out in the last 15 minutes count as promised room (checked while the account row is locked, so two requests at the same moment cannot both pass), and the real size is checked again when the upload is confirmed; a file that does not fit is deleted from storage. Same lock when a deleted file is restored. 8 new tests (`test/integration/files.test.ts`) failed on the old code and pass now.

Not changed: the upload link itself does not fix the file length, because that cannot be tested without real R2 (the local stand-in does not check signatures). The 25 MB limit is enforced when the upload is confirmed, and the nightly job removes unconfirmed uploads after a day. Worth testing on real R2 during the staging drill.

### Dependency vulnerabilities

`npm audit`: **43 findings (2 critical, 14 high) → 15 (0 critical, 1 high)**.

| Done | How |
|---|---|
| Multer 2.0.2 → 2.5.0, body-parser 1.20.4 → 1.20.9, gRPC 1.9 → 1.14.7, PostCSS 8.5.31 → 8.5.32, qs → 6.14.3 | npm `overrides` in the root `package.json` (same major versions, so nothing else changes). A CSV-import test (`import.test.ts`) was added first because Multer reads the uploaded file. |
| vitest 2 → 5, Nest CLI 10 → 12 | Test and build tools only (they never run on the server). All tests, lint, types and builds pass. |

Left, and why:

| Finding | Severity | Why it is left |
|---|---|---|
| zod 3 (memory use on huge arrays) | **high** | Fix is zod 4, which changes validation across the whole API. Exposure is limited by the 1 MB request cap. Plan it with the Nest upgrade. |
| @nestjs/core 10 (injection in output) | moderate | Fix is Nest 11 or 12, a framework upgrade that needs its own testing pass. |
| file-type, uuid, google-gax, firebase-admin chain, exceljs | moderate | Code paths we do not use (Nest's file-type validator; uuid with a caller-given buffer). Cleared by the same upgrades. |
| qs | moderate | Needs qs 6.16+; wait for the Express/Nest upgrade. |

Because the lockfile was rebuilt, run `npm ci` (not `npm install`) on the build server so it installs exactly what was tested.

## 3 · Content Security Policy (`apps/web/src/lib/csp.ts`, `src/middleware.ts`)

- Scripts run only if they carry a new one-time code (nonce) made for that page request, and the scripts those load. Injected scripts, inline `onclick`/`onerror` handlers and `javascript:` links are blocked (tested).
- Also set: no plug-ins (`object-src 'none'`), no `<base>` tricks, forms only to ourselves, pages cannot be put in a frame (`frame-ancestors 'none'`).
- Allowed destinations: our API, Firebase sign-in (Google addresses), Cloudflare R2 file uploads (`*.r2.cloudflarestorage.com`).
- The public booking page (`/b/...`) may still be embedded on a business's own website (`frame-ancestors *`).
- **Switches** (web server environment, see `infra/.env.example`): `CSP_EXTRA_CONNECT` adds allowed addresses (e.g. a custom file domain); `CSP_REPORT_ONLY=true` reports without blocking, for testing a new domain.

**Behaviour change to know about:** a one-time code per request means pages are now built when asked for, not stored in advance. All 26 pages show as "dynamic" in the build. The pages are small, so this costs very little server time, but the landing and legal pages are no longer static files.

**Not yet tested here:** Google sign-in (the pop-up) and real R2 uploads, because there is no Firebase or R2 account yet. The policy allows the addresses Firebase and R2 are documented to use. Test both on staging with `CSP_REPORT_ONLY=true` first; if the browser console shows no "violates the Content Security Policy" lines, switch it off.

Known small items left: the browser asks for `/favicon.ico` and gets a 404 (the app has no icon file). Harmless, but add one before launch.

## 4 · Monitoring: what was built and how to switch it on

Nothing needs a paid service. No request bodies, names, phone numbers or e-mail addresses are ever recorded.

| What | Where | Use |
|---|---|---|
| Alive check | `GET /api/v1/health` | Answers `{"ok":true}` or 503 (database + Redis). Point the uptime monitor at it. |
| Deep check | `GET /api/v1/health/ready` | Database (with connection counts), Redis (memory), file storage, and whether the repeating jobs ran lately. 503 if anything is wrong. Second uptime monitor. |
| Numbers | `GET /api/v1/health/metrics` + header `x-metrics-token` | Answer time (p50/p95/p99) and error rate for the last 5 and 60 minutes, memory, job state, payment-webhook problems. Does not exist unless `METRICS_TOKEN` is set. |
| Push alerts | `ALERT_WEBHOOK_URL` | Sent for: an unexpected server error (500), a scheduled job that fails, a failing deep check, a payment webhook with a wrong signature or that cannot be processed. Same kind at most once per 10 minutes, 30 an hour in all. |
| Still-alive ping | `HEARTBEAT_URL` | The worker pings it every 5 minutes. If the pings stop (worker or Redis down), the outside service alerts you. |

Set up (all free tiers; verify current limits on each site before relying on them):

1. **UptimeRobot** (already in doc 07): monitor 1 = `https://<API_DOMAIN>/api/v1/health` every 5 minutes. Monitor 2 = `.../health/ready`, keyword `"ok":true`.
2. **healthchecks.io**: make a check with a 10-minute period; copy its ping address into `HEARTBEAT_URL`.
3. **Alert address**: a Discord/Slack channel webhook, or a ntfy.sh topic address, into `ALERT_WEBHOOK_URL`. Send a test: stop the worker for 6 minutes, or call the API with a bad Razorpay signature.
4. **Metrics**: put a random 16+ character secret in `METRICS_TOKEN`, then `curl -H "x-metrics-token: <secret>" https://<API_DOMAIN>/api/v1/health/metrics`. For a picture over time, add that address to Better Stack or Grafana Cloud free later; not needed for the pilot.
5. **Errors in code (Sentry)**: not added. The push alert already covers unexpected server errors and failed jobs with the route and message. Add Sentry only once you have an account (it needs a DSN and one more library).

What the alerts mean:

| Alert starts with | Meaning | First thing to do |
|---|---|---|
| `server-error:GET /api/v1/...` | An unexpected error in that route | Look at `docker compose logs api` around that time |
| `job-failed:<name>` | A scheduled job failed (reminders, nightly backup copy, retention) | `docker compose logs worker`; run the job's check by hand |
| `not-ready` | Database, Redis, storage or jobs failing | The message names which one |
| `payment-webhook:bad-signature` | Razorpay's secret in `.env` does not match the dashboard | Fix `RAZORPAY_WEBHOOK_SECRET`, resend the event from Razorpay |
| `payment-webhook:processing-error` | A payment could not be recorded | Check it by hand in Razorpay; Razorpay retries |

Limits to know: the numbers live in the API's memory, so a restart resets them; a process that crashes completely is caught by the uptime monitor and the still-alive ping, not by a push alert.

## 5 · Accessibility

Automated (every CI run): WCAG 2.1 A and AA rules from axe-core on all public pages, every signed-in page and every settings tab, in light and dark themes. Anything "serious" or "critical" fails the build. Two real problems were found and fixed:

- The "I agree to the Terms" box had links inside itself, which screen readers cannot announce properly. The checkbox is now the tick; the text is its label.
- Low contrast: white on teal in the settings tabs (4.17:1, needs 4.5) and the red delete-style buttons in dark mode (2.99:1). The tabs now use a darker tint; the dark-mode buttons use the light red text colour. Both are small visual changes.

Automated checks find about a third of real problems. **Manual pass before launch** (about 1 hour; NVDA is free on Windows, TalkBack is built into Android):

- [ ] Keyboard only: sign up, finish setup, add a customer, upload a file, log out. Focus is always visible; nothing needs the mouse.
- [ ] NVDA (or VoiceOver/TalkBack): each form field is read with its label; error messages are announced when they appear; the sign-up checkbox is read as "I agree to ... checkbox".
- [ ] Zoom the browser to 200%: nothing is cut off or overlaps.
- [ ] Windows "High contrast" mode: text and buttons are still readable.
- [ ] Set the system to "reduce motion": the animations on the login pages stop or reduce.
- [ ] Phone, 360 px wide: every button is easy to tap; no sideways scrolling.

## 6 · Drills that need real servers (not done)

Do these on staging before any customer uses the system, and tick them in [12](12-launch-checklist.md):

- [ ] **Load test**: `infra/load-test/k6.js` against staging (target in doc 07: 300 people online, 95% of answers under 300 ms). Watch `/health/metrics` while it runs.
- [ ] **Restore drill**: `infra/scripts/restore-test.sh`; then `restore-business.sh` for one business (doc 23, G3).
- [ ] **Backup verification**: confirm real backups exist in R2 and the R2 bucket lock is on (doc 23, G5); check a file's second copy.
- [ ] **Standby failover**: `infra/standby/promote.sh`, then point the domain at the standby and back.
- [ ] **Alerts**: confirm a test alert reaches your phone (section 4, step 3).
- [ ] **Real R2 and Firebase**: upload a file; sign in with Google; with `CSP_REPORT_ONLY=true` the console shows no policy violations.

## 7 · Running everything

```bash
npm run db:up                 # Postgres, Redis, file stand-in (Docker Desktop must be running)
npm run migrate
# terminal 1: the API with development login and the monitoring token
METRICS_TOKEN=local-metrics-token-0123456789 AUTH_DEV_BYPASS=true node apps/api/dist/main.js
# terminal 2: the worker
node apps/api/dist/worker.js
npm run test:integration      # 127 API tests (set the same METRICS_TOKEN in this terminal to include the metrics tests)
# terminal 3: the web app, built for development login, started like production
NEXT_PUBLIC_DEV_AUTH=true npm run build -w @vbw/web
CSP_EXTRA_CONNECT=http://localhost:9090 npm start -w @vbw/web
npm run test:e2e              # 28 browser tests in your installed Chrome
```

On a build server set `CI=true` and run `npx playwright install --with-deps chromium` first. The CI file now does all of this (S3 stand-in included). **Not run on GitHub yet**; the first run may need small fixes (browser install, ports).

Docker Desktop on this PC crashed again on the stale `.sock` files (the known problem). Moving `%LOCALAPPDATA%\Docker\run` and `%LOCALAPPDATA%\docker-secrets-engine` aside (they were renamed `*.stale-20261030-*`, nothing deleted) and restarting fixed it.

## 8 · Changes that touch behaviour (for your decision)

None of these change a feature a customer uses, but you asked to be told:

1. **Upload limit is stricter**: uploads that would pass the 4 GB limit are now refused at confirm time too (before, they were accepted).
2. **Pages are rendered on request** because of the nonce (section 3).
3. **Terms checkbox**: the tick is the checkbox and the text beside it is its label; clicking the text still ticks it; keyboard focus now shows on the tick.
4. **Two colour tweaks** for contrast (section 5).
5. **Error responses are unchanged**, but unexpected 500s now also send a push alert if `ALERT_WEBHOOK_URL` is set.

## 9 · Uncommitted work

`git status` shows the whole lead-capture feature (migration 011, `leads.module.ts`, the Leads screen, tests) as uncommitted, and now also today's changes mixed with it. Suggested split, so each commit can be reviewed alone:

1. Lead capture (the files that were already changed or untracked before today).
2. Security and dependencies (`files.module.ts`, `files.test.ts`, `import.test.ts`, `package.json`, `package-lock.json`).
3. Content Security Policy + accessibility fixes (`apps/web/src/...`, `app.css`).
4. Browser tests + CI (`apps/web/e2e/`, `playwright.config.ts`, `.github/workflows/ci.yml`, `eslint.config.mjs`).
5. Monitoring (`common/monitor.ts` and its wiring, env examples, `infra/docker-compose.yml`).
