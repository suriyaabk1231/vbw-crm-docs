# 20 · Handover for a new conversation (as of 27 Oct 2026)

Paste this file (or point Claude at it) at the start of the new conversation. Read it together with `README.md` and `11-decisions-log.md` in this folder.

## 1. What VBW CRM is
- Simple, real-time CRM for Indian businesses. **₹599/month + GST per account**, each account = **2 people** (1 Owner + 1 Member), **20 GB storage per account, shared** (since 27 Oct 2026; trial 100 days for everyone).
- Three launch industries: **Clinics, Gyms, Education centres**. Choosing one at setup loads stages, fields, appointment types, follow-up types, WhatsApp messages and dashboard cards. Screen words change per industry (Patients / Members / Students, Follow-ups / Renewals / Fees).
- **Web first** (Next.js). Android (Expo) later. No payments inside the Android app. Razorpay is phase 2: until then payment is by UPI/bank transfer and a WhatsApp message; the owner activates the account with the `activate` CLI.
- Hosting budget ~₹2–3K/month for the 30-account pilot, about ₹6,200–6,700 for 100 accounts (DigitalOcean Bangalore main server + Contabo Mumbai standby + Cloudflare R2, as in docs 22 and 34; prices from third-party sources, **recheck before buying**).
- Standing rules from the user: **verified answers only** (research, cite, label estimates), **INR**, **new documents go in `vbw-crm-docs`**, code in `vbw-crm`.

## 2. Where things are
| What | Path |
|---|---|
| Code (monorepo) | `C:\Users\Surya\Downloads\car web\vbw-crm` (`packages/core`, `apps/api`, `apps/web`, `apps/mobile`, `infra`) |
| Docs | `C:\Users\Surya\Downloads\car web\vbw-crm-docs` (numbered files 01–35 + README; start with 35 for the full picture) |
| Design handoff | `C:\Users\Surya\Downloads\car web\design\handoff-v2-2026-10-27\design_handoff_vbw_crm\` (README.md, BACKEND.md, SETTINGS_AND_STATES.md, `designs/*.dc.html`) |
| Docker fix script | `C:\Users\Surya\Downloads\car web\fix-docker.ps1` |
| Memory | `C:\Users\Surya\.claude\projects\C--Users-Surya-Downloads-car-web\memory\` |

**Both folders are git repos on `main` (updated 31 Oct 2026).** `vbw-crm` has all code committed; `vbw-crm-docs` holds docs 01–35. There is no GitHub remote yet: the owner pushes. Run `git status` before committing. Commit messages end with the attribution line given for the session.

## 3. Stack
- **Web:** Next.js 15.5 App Router, React 19, TypeScript, plain CSS with variables (`tokens.css`, `glass.css`, `pulse.css`, `app.css`; classes prefixed `v-`), Tailwind 3.4 only lightly (auth pages still use a few old classes `.input`, `.btn-primary` from `globals.css`), Nunito via `next/font/google`, `lucide-react` pinned at 1.54.0, `marked` for legal text.
- **API:** NestJS, zod validation, PostgreSQL with row-level security (`withBusiness`), Redis + BullMQ worker, Firebase Auth (local dev uses a bypass), migrations `001`–`011` in `apps/api/src/db/migrations/`.
- **Core:** `packages/core` (industry templates, labels, constants such as `LEGAL_VERSION`, `MAX_PEOPLE_PER_ACCOUNT`, `RETENTION`, call outcomes, formatters). Rebuild with `npm run build:core` after editing.
- Currency/phone formats: INR (`formatINR`), dates `formatDate`/`formatTime` from `@vbw/core`.

## 4. Complete feature list (all built and working)
### Account, login, security
- Sign-up (business name, your name, agree to Terms/Privacy/DPA checkbox, theme choice remembered), login, forgot/reset password, accept team invite. Firebase Auth in production; dev bypass locally (`NEXT_PUBLIC_DEV_AUTH=true`, headers `x-dev-uid`, `x-dev-email`).
- Device rule: each person on **1 phone + 1 browser**; a new login on the same kind logs the old one out ("You were logged out because this account was used on another device"). API needs `x-device-id` (register via `POST /auth/devices` with `kind` `phone` or `browser`).
- Owner vs Member roles; owner-only endpoints (`@OwnerOnly`). Rate limits (e.g. signup 5/hour).
- Per-business data isolation by PostgreSQL RLS. Access log kept 1 year. Breach procedure documented.
- Terms acceptance stored with the version (`LEGAL_VERSION = 2026-10-24-draft`).

### Compliance gaps G1–G7 (built, DPDP Act 2023)
Account deletion by owner (grace period `RETENTION.accountDeleteGraceDays`, cancellable), purge of deleted contacts, retention job for lapsed accounts (**90-day auto-erasure is switched OFF by the user's decision**), 1-year access log, breach procedure doc, Terms/Privacy/DPA/Sub-processors pages with the sign-up checkbox, grievance officer contact (`lib/contact.ts`, values still empty: `NEXT_PUBLIC_*` env vars).

### Patients / Members / Students
List with search, stage filter, paging; add/edit with industry fields + **custom fields**; duplicate detection by phone/email with **Open existing** or **Merge**; patient page with details, stage change, tags, appointments, follow-ups, open tasks, notes, history timeline, files (upload ≤25 MB), delete (hides record), payment link (only when Razorpay is on); Excel/CSV import with column mapping (duplicates by phone skipped); Excel export of everything.

### Appointments, Follow-ups, Tasks, Pipeline
Day/week calendar; book with type + duration; mark Done / No-show / Cancel / Undo. Follow-ups (renewals/fees/next visits) with views (this week, overdue, upcoming, history), total amount, **Mark paid + schedule next due date**. Tasks (today/overdue/upcoming/done), assigned to a person, linked to a contact. Pipeline board with drag-and-drop on a computer and a **"Move to" menu on phones**; stage names stay as the backend defines them; new stages insert before Won/Lost.

### Calls and WhatsApp
"How did the call go?" (Interested / Call later / Not interested / No answer) with optional reminder task. WhatsApp **only opens a `wa.me` link, the app never sends anything**. Templates with placeholders `{name} {business} {date} {time} {type} {amount} {due_date}`; preview filled via `POST /whatsapp/link` with `preview:true`; owner can add (max 20), edit, delete custom templates.

### Dashboard
Greeting by time of day, 4 industry cards, today's appointments (WhatsApp / Done / No-show), today's tasks (tick to complete), follow-ups due in 7 days (Remind).

### Settings (owner edits, member read-only with an amber note)
Business (profile, locked industry, storage bar, custom fields up to 20), Team & devices (invite by email, copy link, share on WhatsApp, pending invites, remove member, log out device), Stages, WhatsApp messages, Plan & billing, Data (export, activity log, privacy links, delete account with type-the-name + confirm).

### States and notice bars (priority order)
Owner erasure request > lapsed-erasure warning > payment pending > **view-only (plan ended)** > payment failed > trial ending (≤7 days) > paid plan ending (≤7 days). **View-only mode:** API returns 402 `SUBSCRIPTION_REQUIRED` on writes; `/me` returns `business.viewOnly`; buttons are disabled with the tooltip "Renew your plan to make changes"; export still works.

### Look and feel
New design from the Claude Design handoff ("1B" liquid-glass direction on public pages, teal band + sidebar in-app). **Light / Dark / Automatic per person**, saved in `localStorage` key `vbw.theme` and on the server (`users.theme`, `PATCH /me/preferences`); a pre-paint script avoids flashes; low-end phones get `data-glass="flat"`. Public pages: landing page, login/sign-up/forgot/reset/invite with the animated "VBW Pulse" (`lib/pulse.ts`), reduced-motion respected. Legal pages restyled (draft bar, document pills, teal clause numbers).

## 5. Backend changes made this project
Theme preference; WhatsApp templates create/delete; WhatsApp link `preview`; History filter for follow-ups; stage insertion before Won/Lost; "payment pending" state (`paymentPendingAt`, set when the owner says they paid); data retention + deletion jobs; access log; terms acceptance. The BACKEND.md in the handoff did not match the real stack, so each need was mapped onto existing endpoints instead of rewriting.

## 6. How to run it locally
1. Docker Desktop up; `npm run db:up` (Postgres + Redis). If Docker crashes with `sailor-ingest.sock`, run `fix-docker.ps1` (cause: leftover socket files after an unclean shutdown).
2. From `vbw-crm`: `npm run build:core`, `npm run migrate`, `npm run dev:api` (port 4000, health: `/api/v1/health`), `npm run dev:web` (Claude preview config `vbw-web` uses port 3100).
3. Dev login in the browser console: `localStorage.setItem('vbw.devUser', JSON.stringify({uid:'u1', email:'a@test.local'}))`, then open `/app`. Create the account once through `POST /api/v1/auth/signup` (body needs `acceptedTerms: "2026-10-24-draft"`), register a device, then the setup wizard runs.
4. Tests: `npm test` (11 core + 19 API, all passing). Build: `npm run build -w @vbw/web` (passes). Type-check: `npx tsc --noEmit` in `apps/web`.
5. Clean test data: `delete from businesses where id = '…'` in Postgres (`docker exec vbw-crm-dev-postgres-1 psql …`).

## 7. Known gotchas
- Never run `next build` while the dev server runs (clobbers `.next`); stop the preview first.
- Clicking the agree checkbox by ref hits the DPA link; tick it with `document.querySelector('[role=checkbox]').click()`.
- Preview pane renders only after a screenshot; take a second one if animations look unplayed.
- `lucide-react` 1.54.0 has no `ClipboardUser`; use `ClipboardList`.
- The round "N" at the bottom-left of the dev preview is Next's dev badge, not part of the app.
- Windows machine: no `python`; use bash/PowerShell/Node.

## 8. What is NOT done (next conversation)
**Code**
1. `git init` and first commit (nothing is under version control).
2. Screens I only built and type-checked but did not click through: Add-patient duplicate/Merge dialog, Book appointment, Add follow-up, Add task, Call outcome dialog, Mark paid, payment link, file upload/delete, import mapping, stage rename/delete, message edit/new/delete, member view of Settings, inviting a second person, Excel download. Also the three industries (only Clinic was tried), reduced motion, long-name wrapping everywhere, tablet width, and a real phone.
3. Auth pages still import a few old Tailwind classes; remove when convenient.
4. Android app (`apps/mobile`) not restyled or tested against the new backend.
5. Razorpay (subscriptions, payment links) phase 2. Brevo/SES branded reset emails not connected.
6. Accessibility pass (contrast of teal on dark, focus rings, screen-reader labels) and a real security review before launch.

**User-side (cannot be done by Claude)**
- Create Firebase, Brevo, Cloudflare accounts and keys; buy/set up servers; verify the hosting prices.
- Lawyer review of the Terms/Privacy/DPA draft (doc 17); fill in legal entity name, registered address, grievance officer name/email/phone, support email/WhatsApp.
- Decide whether to switch the 90-day auto-erasure on.
- Test on a real Android phone; record the demo video (doc 14) and design the brochure (doc 15); find pilot clients with the call script (doc 13) and target list (doc 16).

## 9. Docs index (in this folder)
09 go-to-market · 10 operations · 11 decisions log · 12 launch checklist · 13 call script (English + Tamil) · 14 demo video script · 15 brochure copy · 16 target list template · 17 Terms/Privacy draft for the lawyer · 18 Claude Design prompts · 19 login pages design prompt · **20 this handover**. (Docs 01–08 are the original plan/architecture files listed in README.md.)

## 10. Suggested first message in the new conversation
"Read `vbw-crm-docs/20-handover-new-conversation.md`. Start with git init + first commit, then click-test every screen listed in section 8 on phone and desktop in light and dark, fix what breaks, and report."
