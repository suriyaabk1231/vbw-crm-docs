# Plan lengths, calendar dates and "Due soon" — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Let an owner choose "after N days/months" or a calendar date for a renewal or follow-up, get several reminders before it, and see everyone needing action on a separate "Due soon" screen (web and Android).

**Architecture:** Extend the existing `renewals` table with four additive columns. Pure date and reminder rules live in `packages/core` (tested once, used by API and web). The API gains `GET /due-soon` and a snooze endpoint, and the renewal create/update/renew routes accept the new fields. A new worker job pushes one message per reminder point. Industry defaults live in the industry templates (stored in the database; each changed template needs a `version` bump).

**Tech Stack:** TypeScript, NestJS + zod + `pg` (API), Next.js 15 / React 19 (web), Expo / React Native (Android), Vitest, Playwright.

**Spec:** `vbw-crm-docs/36-design-plans-and-due-soon-reminders.md` (approved 31 Oct 2026). Repo root for paths below: `C:\Users\Surya\Downloads\car web\vbw-crm`.

## Global Constraints

- **Do not change** the Renewals screen and its views, `GET /renewals` and its view filters, the 09:00 `renewal-alerts` summary push, payments, the Developer API, the Excel export, the dashboard cards. Existing renewal rows (no new fields) behave exactly as today.
- Migration `012` is **additive only**, with defaults, so old rows keep working.
- "Today" is Indian time (`Asia/Kolkata`); the **server** computes "after N days/months". Months clamp to month end (31 Jan + 1 month = 28 Feb, 29 in a leap year).
- `remindDays`: up to **6** whole numbers, each **-30 to 365**, no repeats. Positive = days before, negative = days after (`-1` = one day after). Choices offered in forms: **1, 2, 3, 5, 7, 15, 30, 60, 90** before, plus **1 day after**.
- A reminder point earlier than the day the item was **created** is skipped; if several points were missed, **only the latest missed one fires**; each fires **once**.
- New screen name: **"Due soon"**, route `/app/due-soon`, API `GET /api/v1/due-soon`. Android: a **bell with a count on Home**, not a sixth tab.
- Push text contains the person's name, type and date only, **never the note**. A plan-ended (view-only) account gets no new reminders. No messages go to customers.
- Both people may use everything; Android is **type-checked only** (not run on a phone). Commit messages end with the attribution line given for the session.

## Review Focus

1. **Month end:** 31 Jan + 1 month, 30 Aug + 6 months, and 29 Feb in a leap year give valid, clamped dates (Task 1, Task 2 tests).
2. **A reminder point before the item existed** (a 5-day follow-up with a 7-day point): skipped, never fires (Task 1, Task 3 tests).
3. **Garbage input:** duplicate, out-of-range, fractional or more than six `remindDays`; both `dueDate` and `after` sent; neither sent; a past `dueDate` on create; a snooze date in the past → 400 (Task 2 tests).
4. **Gone items:** a renewed, cancelled, snoozed, deleted-contact or other-business item must not appear in Due soon or get a push (Task 2, Task 3 tests).
5. **Clock edge:** at 23:30 Indian time the UTC date is still "yesterday"; "today" must still be the Indian date (Task 1 test).

---

### Task 1: Core rules and industry defaults

**Files:**
- Create: `packages/core/src/plans.ts`, `packages/core/src/plans.test.ts`
- Modify: `packages/core/src/index.ts` (export), `packages/core/src/templates.ts` (types, five templates, version bumps)

**Interfaces:**
- Produces (all exported from `@vbw/core`):
  - `type PlanUnit = 'days' | 'months'`; `interface PlanLength { length: number; unit: PlanUnit }`
  - `REMIND_CHOICES: readonly number[]` = `[1,2,3,5,7,15,30,60,90]`; `MAX_REMIND_POINTS = 6`
  - `todayIndia(now?: Date): string` → `YYYY-MM-DD` in Asia/Kolkata
  - `addPlan(fromISO: string, plan: PlanLength): string` (clamps month end)
  - `normalizeRemindDays(raw: unknown): number[] | null` → sorted descending, unique; `null` when invalid
  - `reminderPointToFire(i: { dueDate: string; createdDate: string; remindDays: number[]; remindedDays: number[]; today: string }): number | null` → the latest reached, unsent, not-before-creation point, or `null`
  - `isDueSoon(i: { dueDate: string; createdDate: string; remindDays: number[]; today: string; snoozedUntil: string | null }): boolean` → true once any usable point is reached, not snoozed
  - `daysUntil(dueDate: string, today: string): number` (negative when overdue)
  - On `IndustryTemplate.renewalTypes[]` add optional `plan?: { unit: PlanUnit; lengths: number[]; defaultLength?: number }` and `remindDays?: number[]` (defaults in the table of spec §5.3).

- [ ] **Step 1: Write failing tests in `plans.test.ts`:** `addPlan('2026-01-31', {1,'months'})` → `'2026-02-28'`; `addPlan('2028-01-31', {1,'months'})` → `'2028-02-29'`; `addPlan('2026-08-30', {6,'months'})` → `'2027-02-28'`; `addPlan('2026-12-04', {12,'days'})` → `'2026-12-16'`; `todayIndia(new Date('2026-10-30T18:29:00Z'))` → `'2026-10-30'` and `todayIndia(new Date('2026-10-30T18:30:00Z'))` → `'2026-10-31'` (midnight in India is 18:30 UTC); `normalizeRemindDays([3,7,1,3])` → `null`, `[7,3,1,-1]` stays `[7,3,1,-1]`, `[0.5]`, `[400]`, `[-31]`, 7 items, `'3'` → `null`; `reminderPointToFire` cases: due in 3 days with `[7,3,1]` created 20 days ago → `7`; with `remindedDays [7]` → `3`; missed 7 and 3, today 1 day before → fires only `1`; created today for a 5-day follow-up with `[7,3]` → `3` only when 3 days before and never `7`; negative point `-1` fires the day after; `isDueSoon` false before first point, true after, false while snoozed, true again on the snooze date.
- [ ] **Step 2: Run** `npm test -w @vbw/core` → expect FAIL (module missing).
- [ ] **Step 3: Implement `plans.ts`** with the signatures above; date arithmetic on `Date.UTC` parts (no local-time `Date` methods); `todayIndia` via `Intl.DateTimeFormat('en-CA', { timeZone: TIMEZONE })`.
- [ ] **Step 4: Run** `npm test -w @vbw/core` → all pass.
- [ ] **Step 5: Add template defaults** exactly per spec §5.3 (gym lengths 1–18 months; hospital 3,5,7,12,18,30 days; education 1,2,3 months; retail 7,15,30,60,90 days; real estate agreement 11 months, follow-up 2,5,7 days), set `remindDays`, and bump `version` of hospital, gym, education, retail, real_estate by one. Add a test that every template's `remindDays` passes `normalizeRemindDays` and every `lengths` is a non-empty list of positive integers.
- [ ] **Step 6: Run** `npm run build:core && npm test -w @vbw/core` → pass. Commit: `git add packages/core && git commit -m "Add plan date math, reminder rules and industry defaults"`.

### Task 2: Database and renewal API

**Files:**
- Create: `apps/api/src/db/migrations/012_renewal_plans.sql`, `apps/api/test/integration/due-soon.test.ts`
- Modify: `apps/api/src/modules/schedule.module.ts` (`renewalInput`, `createRenewal`, `updateRenewal`, `setRenewalStatus`, controller), `apps/api/test/integration/flows.test.ts` only if an existing assertion breaks

**Interfaces:**
- Consumes: `normalizeRemindDays`, `addPlan`, `todayIndia`, `PlanLength`, `isDueSoon`, `daysUntil` from Task 1.
- Produces (HTTP):
  - `POST /renewals` body adds: exactly one of `dueDate` (`YYYY-MM-DD`, not before today) or `after: { length: 1..999, unit }`; optional `remindDays: number[]`. Response unchanged (`{id}`) plus `dueDate`.
  - `PATCH /renewals/:id` accepts the same optional fields.
  - `POST /renewals/:id/status` body adds optional `nextAfter: PlanLength`, `nextRemindDays`; the next item **copies** `plan_length`, `plan_unit`, `remind_days` from the old one unless overridden; response keeps `nextRenewalId`.
  - `GET /due-soon` → `{ count: number, items: DueSoonItem[] }`, `DueSoonItem = { id, contactId, contactName, contactPhone, type, dueDate, daysLeft, planText: string|null, amountPaise, notes, remindDays }`; sorted by `dueDate` ascending; only `status = 'due'`, contact not deleted, `isDueSoon` true.
  - `POST /renewals/:id/snooze` body `{ until: 'YYYY-MM-DD' }` (after today, at most 90 days ahead) → `{ ok: true }`.

- [ ] **Step 1: Write failing integration tests** in `due-soon.test.ts` (helpers: `createAccount`, `addMember`, `addContact`, `call`, `db`, `setTrialEnd`): create with `after {3,'months'}` returns the date equal to core `addPlan(todayIndia(), …)`; create with `dueDate`; 400 for: both `dueDate` and `after`, neither, past `dueDate`, `remindDays` `[3,3]`, `[0.5]`, seven items, `after.length` 0 or 1000; an item with default points appears in `GET /due-soon` only after its due date is moved (via `db()`) inside the window, with `daysLeft` and `planText` `"3 months"`; a point before creation is ignored; snooze hides it and the same item returns when `snoozed_until` is moved to today; snooze in the past → 400; renew with `nextAfter` creates a new item with copied points and the old one disappears from Due soon; cancelling removes it; a soft-deleted contact's item is absent; member can read and snooze; other business sees `count: 0` and gets 404 on snooze of the first business's item; a lapsed account (`setTrialEnd(c,'past')`) can `GET /due-soon` but gets 402/403 on snooze.
- [ ] **Step 2: Run** `METRICS_TOKEN=local-metrics-token-0123456789 npx vitest run --config vitest.integration.config.ts test/integration/due-soon.test.ts` in `apps/api` (API and worker running, `npm run migrate` done) → FAIL.
- [ ] **Step 3: Migration `012_renewal_plans.sql`:** `ALTER TABLE renewals ADD COLUMN plan_length int CHECK (plan_length BETWEEN 1 AND 999), ADD COLUMN plan_unit text CHECK (plan_unit IN ('days','months')), ADD COLUMN remind_days int[] NOT NULL DEFAULT '{}', ADD COLUMN reminded_days int[] NOT NULL DEFAULT '{}', ADD COLUMN snoozed_until date;` and a CHECK that `plan_length` and `plan_unit` are both null or both set; index `(business_id, status, due_date)` already exists.
- [ ] **Step 4: Implement** the zod changes and the SQL: compute `after` in SQL as `(now() AT TIME ZONE 'Asia/Kolkata')::date + make_interval(days|months => $n)`; validate `remindDays` with `normalizeRemindDays` (400 on `null`); extend `setRenewalStatus` insert to copy the three plan columns; `GET /due-soon` selects open renewals and filters with `isDueSoon` in code (list is small); snooze updates `snoozed_until`. Keep every existing query and route untouched.
- [ ] **Step 5: Run** the new test file, then the whole suite `npm run test:integration` from the repo root → all pass (existing 145 included).
- [ ] **Step 6: Commit:** `git add apps/api && git commit -m "Add renewal plans, reminder points, Due soon list and snooze"`.

### Task 3: Reminder push job

**Files:**
- Create: `apps/api/src/common/plan-reminders.ts`, `apps/api/test/integration/plan-reminders.test.ts`
- Modify: `apps/api/src/jobs/jobs.service.ts` (`SCHEDULES`, `run()`, new method)

**Interfaces:**
- Consumes: `reminderPointToFire`, `todayIndia` (Task 1); columns from Task 2.
- Produces: `collectPlanReminders(c: PoolClient, today: string): Promise<{ businessId: string; renewalId: string; contactName: string; type: string; dueDate: string; point: number }[]>` — for every paid-up business's open item with a due point, **appends the point to `reminded_days` in the same transaction** and returns what to push. `SCHEDULES['plan-reminders'] = { pattern: '5 9 * * *' }`.

- [ ] **Step 1: Write failing tests** (import the function from `../../src/common/plan-reminders`, use `db().connect()` with the owner login): item at its first point returns one entry and a second call returns none; a later point fires after the first was recorded; an item snoozed, cancelled, renewed or for a deleted contact returns none; an account with the trial ended returns none; points missed together fire only the latest; the note text is not in the returned object.
- [ ] **Step 2: Run** the test file → FAIL.
- [ ] **Step 3: Implement `collectPlanReminders`** (`SELECT … FROM renewals JOIN contacts … JOIN businesses` with the same paid-up condition the `automations` job uses; decide with `reminderPointToFire`; `UPDATE renewals SET reminded_days = array_append(reminded_days, $2)`), and `JobsService.planReminders()` that calls it inside `this.db.adminTx`, then pushes to the business's people via the existing private `push`, grouping more than five per business into "<first name> and N more are due soon". Leave `renewalAlerts` as it is.
- [ ] **Step 4: Run** the new tests and `npm run lint` → pass. Confirm the worker log shows `Worker started with 15 scheduled jobs`.
- [ ] **Step 5: Commit:** `git add apps/api && git commit -m "Add the Due soon reminder push job"`.

### Task 4: Web — form, renew, client page, Due soon

**Files:**
- Create: `apps/web/src/app/app/due-soon/page.tsx`, `apps/web/src/components/app/PlanPicker.tsx`, `apps/web/e2e/due-soon.spec.ts`
- Modify: `apps/web/src/components/app/dialogs.tsx` (`FollowUpDialog`, `MarkPaidRow`), `apps/web/src/components/app/Shell.tsx` (nav item + count), `apps/web/src/app/app/renewals/page.tsx` (pass plan to `MarkPaidRow`), `apps/web/src/app/app/contacts/[id]/page.tsx` (show plan text), `apps/web/src/lib/types.ts` (`Renewal` gains `planLength`, `planUnit`, `remindDays`; add `DueSoonItem`), `apps/web/e2e/accessibility.spec.ts` (add `/app/due-soon`)

**Interfaces:**
- Consumes: Task 2 API; `addPlan`, `todayIndia`, `REMIND_CHOICES`, template `renewalType.plan` / `remindDays`.
- Produces: `PlanPicker({ type: RenewalTypeDef; value: PlanPick; onChange: (v: PlanPick) => void })` where `PlanPick = { mode: 'after' | 'date'; length: number; date: string; remindDays: number[] }` and a helper `planBody(pick: PlanPick): { after?: PlanLength; dueDate?: string; remindDays: number[] }`.

- [ ] **Step 1: Write failing browser tests** in `due-soon.spec.ts` (`newAccount`, `apiAsBrowser`, `uniq`): gym account — add a membership "After 3 months" and see the resulting date in words, the item appears in Renewals; hospital account — pick a calendar date and the 3-days default is pre-selected; renew with a longer length shows "Upgrade" on the longer chips; an item moved into its window via the API (set through `db()`) shows on `/app/due-soon` with Call, WhatsApp, Renew, Snooze and Not renewing, and the nav shows the count; Snooze removes it; Not renewing removes it; a plain renewal made with the old API shape still works.
- [ ] **Step 2: Run** `npx playwright test e2e/due-soon.spec.ts` in `apps/web` → FAIL.
- [ ] **Step 3: Implement** `PlanPicker` (two radio choices "After" with industry length chips, "Pick a date" with `<input type="date">`; shows `formatDate` of the result; "Remind me" multi-select chips from `REMIND_CHOICES` plus "1 day after"), use it in `FollowUpDialog` and `MarkPaidRow` (pre-select the current length; mark longer chips "Upgrade"); the Due soon page lists `GET /due-soon` as cards (person, type and plan, date with "in 3 days / 2 days overdue", the note) with the five buttons, reusing `WhatsAppDialog` with the `renewalId` and a `tel:` link for Call; the nav item "Due soon" (icon `BellRing`) with a count fetched once on mount and after changes. Keep every existing prop and behaviour of both components when the new fields are absent.
- [ ] **Step 4: Run** the new spec, then the full browser suite `npm run test:e2e` (accessibility scan includes `/app/due-soon`, light and dark) and `npx tsc --noEmit -p apps/web` → all pass (existing 32 included).
- [ ] **Step 5: Commit:** `git add apps/web && git commit -m "Add plan picker, upgrade at renewal and the Due soon screen on web"`.

### Task 5: Android — form, Due soon, Home bell

**Files:**
- Create: `apps/mobile/app/due-soon.tsx`, `apps/mobile/src/plan.tsx` (picker)
- Modify: `apps/mobile/app/contact/[id].tsx` (renewal sheet near line 343), `apps/mobile/app/(tabs)/index.tsx` (bell with count), `apps/mobile/app/(tabs)/renewals.tsx` (renew uses the picker), `apps/mobile/app/_layout.tsx` (register the route)

**Interfaces:**
- Consumes: Task 2 API; core helpers and template fields.
- Produces: `PlanPicker` for React Native with the same `PlanPick` shape and `planBody` helper as the web (re-exported from core if shared, otherwise duplicated by name).

- [ ] **Step 1: Implement** the picker (chips for lengths, a date entry using the existing date input pattern in `contact/[id].tsx`), the Due soon screen (cards with Call via `Linking.openURL('tel:')`, `WhatsAppSheet` with `renewalId`, Renew / Upgrade, Snooze, Not renewing), and the Home bell with the count from `GET /due-soon`.
- [ ] **Step 2: Run** `npx tsc --noEmit -p apps/mobile` → no errors; `npm run lint` from the repo root → clean.
- [ ] **Step 3: Commit:** `git add apps/mobile && git commit -m "Add plan picker and Due soon screen on Android (type-checked only)"`.

### Task 6: Documentation, full verification, hand-off

**Files:**
- Create: `vbw-crm-docs/38-due-soon-reminders-built.md`
- Modify: `vbw-crm-docs/README.md`, `vbw-crm-docs/35-complete-system-overview.md` (routes, job, tables columns, screens, test counts), `vbw-crm-docs/11-decisions-log.md`

- [ ] **Step 1:** Run everything: `npm run lint`, `npm run typecheck`, core and API unit tests, `npm run test:integration`, `npm run test:e2e`. Record the counts.
- [ ] **Step 2:** Write doc 38 (what was built, how to use it per industry, the defaults table, honest limits: push untested with real Firebase, Android not run on a phone, defaults are vendor patterns), update the three docs above.
- [ ] **Step 3:** Check in the browser pane: add a gym membership by months, add a dental follow-up by calendar date, open Due soon, take screenshots at desktop and phone width. Report anything that looks wrong.
- [ ] **Step 4: Commit both folders** (`vbw-crm`, `vbw-crm-docs`) after the owner agrees.

## Self-review

- **Spec coverage:** length or calendar date (T1, T2, T4, T5); several points plus after-date point (T1, T2); skip before creation, latest missed only, once only (T1, T3); Due soon screen with five actions (T2, T4, T5); push (T3); upgrade and copy to next item (T2, T4); note on the card, not in the push (T3, T4); per-industry defaults and template version bumps (T1); unchanged features (Global Constraints, T2 Step 5, T4 Step 4); view-only and isolation (T2); docs and decisions (T6). No gaps found.
- **Types and names:** `PlanLength`, `PlanPick`, `planBody`, `reminderPointToFire`, `isDueSoon`, `collectPlanReminders`, route names and column names match across tasks.
- **Proportion:** the plan states signatures, values and tests; bodies are left to the implementer.
