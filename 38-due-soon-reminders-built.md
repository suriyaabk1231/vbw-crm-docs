# 38 — Plan lengths, calendar dates and "Due soon": what was built

*Built 31 Oct 2026 from the approved design ([36](36-design-plans-and-due-soon-reminders.md)) and plan ([37](37-plan-due-soon-reminders.md)). "Built" means built and tested on the local copy. Nothing here has run on a real server, with real Firebase push, or on a phone.*

## 1. What an owner can do now

1. **Add a follow-up, renewal or membership** and choose **"After…"** a length (3 months, 12 days) **or "Pick a date"** on a calendar (for example 16 December). The screen shows the resulting date in words before saving.
2. Choose **"Remind me"** points (1, 2, 3, 5, 7, 15, 30, 60, 90 days before, and 1 day after). Each industry starts with its own defaults (section 3).
3. Open **Due soon** (new menu entry, with a count) to see everyone who needs action now, with the person's note, the plan and "Due in 3 days / 2 days overdue".
4. On each card: **Call**, **WhatsApp**, **Renew / Upgrade**, **Snooze** (tomorrow, 3 days, a week, or a date up to 90 days away), **Not renewing**.
5. **Renew / Upgrade** opens the same picker with the current length selected; longer lengths are marked **"Upgrade"** (gym: up to 18 months). The next item keeps the length and reminder points unless changed.
6. A **push** goes to everyone on the account at 09:05 when a reminder point is reached ("Asha: Membership is due in 3 days"). More than five in one day are grouped. The note is never in the push.

Everything else (the Renewals screen and its views, the 09:00 summary push, payments, the Developer API, the Excel export, the dashboard cards) works as before. An old-style renewal (just a date) still works.

## 2. Where it is

| Part | Where |
|---|---|
| Date and reminder rules (tested) | `packages/core/src/plans.ts` |
| Industry lengths and default reminder points | `packages/core/src/templates.ts` (`renewalTypes[].plan`, `.remindDays`; the five templates' `version` went up by one) |
| Database (migration 012) | `renewals`: `plan_length`, `plan_unit`, `remind_days`, `reminded_days`, `snoozed_until` |
| API | `apps/api/src/modules/schedule.module.ts`: `POST/PATCH /renewals` (length or date, reminder points), `POST /renewals/:id/status` (upgrade), `GET /due-soon`, `POST /renewals/:id/snooze` |
| Reminder push | `apps/api/src/common/plan-reminders.ts` and the `plan-reminders` job in `jobs.service.ts` (09:05) |
| Web | `components/app/PlanPicker.tsx`, `dialogs.tsx` (add form, `MarkPaidDialog`), `app/app/due-soon/page.tsx`, menu entry in `Shell.tsx` |
| Android | `src/plan.tsx`, `src/sheet.tsx`, `app/due-soon.tsx`, bell with a count on Home, the add form on the client screen |

## 3. Defaults per industry (editable per item)

Lengths come from the industry; a date can always be picked instead. Reminder points are pre-selected and can be changed in the form.

| Industry | Type | Lengths | Default reminders |
|---|---|---|---|
| Gym | Membership | 1 to 18 months | 7, 3, 1 days before; 1 day after |
| Hospital (dental and others) | Next visit | 3, 5, 7, 12, 18, 30 days | 3 days before |
| Education | Fee instalment | 1, 2, 3 months | 3 days before; 1 day after |
| Retail | Repeat purchase | 7, 15, 30, 60, 90 days | 3 days before |
| | Service / warranty due | a date | 7, 3 days before |
| | Balance payment | a date | 3 days before; 1 day after |
| Real estate | Rent agreement renewal | 11, 12, 24, 36 months (11 default) | 90, 60, 30 days before |
| | Follow-up | 2, 5, 7 days | 1 day before |
| | Payment instalment | a date | 3 days before |

**Where these came from:** vendor blogs (gym 7/3/1 and after-expiry, dental per-patient intervals, retail day 7 and day 60, lease 90/60/30), not measured results. For real estate, the real deadline is the notice period in each registered lease deed: edit the points per agreement. **For dental**, use the note field for a short instruction such as "bring the X-ray"; it shows on the Due soon card.

## 4. Rules worth knowing

- **Only the owner's phones and screen are reminded, not customers.** A customer gets a message only when the owner taps the WhatsApp button.
- **A reminder point is sent once.** If several were missed (server off, owner away), only the latest is sent, and the earlier ones are not sent late.
- **A point earlier than the day the item was added is skipped** (a 5-day follow-up never gets "7 days before").
- **Changing the date or the reminder points restarts the reminders** (nothing sent yet, nothing snoozed).
- **Dates are worked out by the server in Indian time.** Months stop at the end of a shorter month (31 Jan + 1 month = 28 Feb).
- **Not renewing** keeps the item in the history as cancelled. **Snooze** brings it back on the chosen day. Accounts whose plan has ended get no reminders and cannot change items.

## 5. Tests (run 31 Oct 2026)

| Suite | Result |
|---|---|
| Core unit tests | 70 passed (23 new: date math, month ends, clock edge at midnight India, reminder rules, a bug fix, industry defaults, helpers) |
| API unit tests | 29 passed |
| API integration tests | 170 passed (25 new: create by length or date, validation, Due soon, snooze, upgrade, isolation, plan ended; reminder collection) |
| Browser tests | 39 passed (7 new), including accessibility scans of Due soon in light and dark |
| Lint and type checks | clean for API, web, Android and core |

**A bug found by the tests and fixed before release:** after the latest of several missed reminder points was sent, the earlier missed points would have been sent later. A point now counts as done once it or any closer point was sent.

**Checked by eye in the browser:** the Due soon screen at desktop and phone width, and the renew panel with "Upgrade" marks.

## 6. Not done, and honest limits

- **The 09:05 push has never been delivered for real:** it needs a Firebase account (not created). Its selection logic is tested.
- **The Android screens are type-checked only**, not run on a phone. The app has no code that opens a screen when a push is tapped (it never had), so a tapped "Due soon" push opens the app, not the screen.
- **No owner-editable length lists and no separate start date** in this version.
- **Real estate reminders for tenants or landlords are not sent**, only to the owner.
- **Defaults come from vendor blogs.** Watch what the pilot actually uses and adjust them in `templates.ts` (remember to raise the template `version`).
- The hourly job did not run on its clock here; the same functions are covered by the tests.

## 7. Security review of this feature (2 Nov 2026)

A hand review of the Due soon code (the built-in `/security-review` skill needs a Git remote, which the repo does not have). **Not an independent test.**

| Check | Result |
|---|---|
| Who can read or change items | Same rules as renewals: both people of a business, nobody else. A person from another business gets an empty list and a 404 on snooze (tested). A view-only account can read but not change (tested) |
| SQL | Every value is a parameter; the reminder selection and the snooze use no text built from input |
| Input size | Reminder points are checked for length before anything else (at most 6); a plan length is at most 999; the 1 MB request limit applies |
| The push | Contains the person's name, the type and a day count only; the note is never sent (tested). It goes only to devices of that business |
| The note and name on screen | Shown as plain text by React (not as HTML). The `tel:` link is built with the `tel:` prefix, so a stored phone value cannot become a script link |
| The job | Works across businesses with the owner login, but only reads open items of paid-up businesses and writes only the "reminder sent" list. A point is recorded as sent before the push, so a failed push is not retried (it could be lost, never duplicated) |
| **Found and fixed: impossible dates** | A date such as `2026-99-99` passed the format check, then the database refused it, giving a 500 error and a team alert. The shared `isoDate` rule now accepts real calendar days only, for renewals, snooze, sales dates and the Developer API (unit and integration tests) |
| Known limit | The Due soon list reads at most the 500 oldest open items with reminders. A business with more than 500 never-actioned overdue items could see newer ones missing. Not a risk at pilot size; clear old items with "Not renewing" |
| Known limit | The list is requested on every page change (for the count in the menu). Fine for 100 accounts; cache it if the load test shows a problem |

**Dependency audit (2 Nov 2026):** 22 findings in production dependencies (21 moderate, 1 high, 0 critical), up from 15 on 30 Oct because new advisories were published; no dependency was added to the server. 15 of them sit inside the Firebase and Excel libraries. The one **high** is `zod` (can use a lot of memory on huge lists); the fix is the major version 4, which was deliberately not taken before launch, and the 1 MB request limit already caps the size of any list. The `@nestjs/core` finding needs Nest 11. Plan: upgrade zod and Nest together after the pilot starts, with the 171 API tests as the safety net.
