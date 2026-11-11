# 33 — Fixed reports and built-in automations

*Built on 30 Oct 2026, after the owner's go-ahead on points 1 and 4 of the priority list.*

## 1. Reports (new menu entry "Reports", web)

Four periods: 7 days, 30 days, 90 days, 1 year. Open to both people. **Excel** download is owner-only (like the existing data export) and is written to the activity log.

| Section | What it shows |
|---|---|
| Top cards | People added in the period; how many of them are **Won** / **Lost** now; how many people moved to Won in the period (whenever they were added) |
| Where they came from | Per source: added, won, won % (people added in the period who are in a Won stage now) |
| What each person did | Calls, notes, WhatsApp messages, stage moves, tasks done in the period; tasks overdue now. Software actions (booking page, API) are not counted as a person |
| In each stage now | Number of people in each pipeline stage, in order, empty stages included |

**Read this when looking at "won %":** it counts people *added in the period* who are won *today*. A 7-day view will look low because recent people have not had time to buy. Use 90 days or 1 year for a fair number.

**Not built (on purpose):** a report builder, cohort/retention charts, campaign ROI, scheduled report emails. Reasons: small shops rarely use a builder; ROI needs ad spend we do not have; scheduled emails need more setup. Revisit after the pilot.

Code: `apps/api/src/modules/reports.module.ts`, `apps/web/src/app/app/reports/page.tsx`.

## 2. Automations (Settings → Automations, owner only)

A fixed set of rules, each a switch. **Everything is off** for every account until the owner turns it on, so nothing changes for anyone who does not open the screen.

| Rule | What it does |
|---|---|
| Hot lead → call task | A person who scores Hot (doc 30) gets a task "Hot lead: call …". Not again for the same person for 7 days, even if the task was finished |
| Quiet lead → follow-up task | A person still being won with nothing logged (call, note, message) for N days (3 to 60, default 7) gets "Follow up: …". Counted from the last thing logged, or from when they were added. Not again within N days |
| Who gets the tasks | The owner (default) or the second person. Also decides who gets the **call-back task of every new lead** from a lead address or the booking page (before, always the owner). If the chosen person is not on the account, the owner gets it |

How it runs: an hourly job (`automations`, at 10 minutes past) for every business that has a rule on and a paid-up plan (a view-only account gets no new tasks), plus **Run now** on the screen. At most **20 tasks per rule per run** so a first run on a big list cannot flood the Tasks screen. A task is a normal task: it shows on Tasks and Home and sends the usual reminder.

**No messages are sent to customers by automations.** The "send a WhatsApp" part of the original example is not included: wa.me links need a person to tap Send, and the WhatsApp Business API is not built (see the priority list).

Code: `packages/core/src/automations.ts` (rules, tested), `apps/api/src/common/automations.ts` (the run), `apps/api/src/modules/automations.module.ts`, `apps/web/src/app/app/settings/AutomationsTab.tsx`.

## 3. Lead sources (point 3): checked, nothing new built

Doc 30 already reads IndiaMART-style pushes and Meta / Google Ads lead shapes through lead addresses. What is left needs things outside the code:

| Item | Why not built |
|---|---|
| Direct Meta Lead Ads | Meta only sends a lead id; fetching the details needs an approved Meta app with business verification and the `leads_retrieval` permission. Cannot be built or tested without those |
| IndiaMART exact field names | I searched again on 30 Oct: IndiaMART's help page shows only the OTP set-up and the reply code, **not the lead field names**. The reader still accepts the common names; the first real lead may need a one-line fix (it will show under "See what arrived") |
| 99acres / MagicBricks / Housing | They email leads; reading email needs an inbound-mail service (not chosen, cost unchecked) |

Per the priority list ("only the one pilots ask for"), the next step is to see which source a pilot actually uses and test that one live.

## 4. A bug found on the way (fixed)

The API's browser rules allowed only GET, POST, PATCH and DELETE. Anything the website sends with **PUT** failed with "Cannot reach VBW CRM". The only existing PUT was **saving a Developer API webhook** in Settings, so that save would have failed in a real browser (the API tests missed it because they do not use a browser). `PUT` is now allowed for our own website only; a test checks that another website is still refused.

## 5. Tests

| Test | Count | What |
|---|---|---|
| `packages/core/src/automations.test.ts` | 6 | Settings reading (rubbish, out-of-range days, `"10"` as text), task titles |
| `apps/api/test/integration/automations-reports.test.ts` | 17 | Settings (defaults, validation, member refused, per-business), hot task (once, assignee, not after done, not when off or won), quiet task (once, 20-task cap), new-lead assignee, plan ended, all report numbers, periods, export permission, isolation, CORS |
| `apps/web/e2e/reports-automations.spec.ts` | 3 | Reports page and period; Automations save/reload/Run now; day count out of range |
| Accessibility scan | — | `/app/reports` and the Automations tab added to the existing light/dark scans |

All suites pass: 47 core, 145 API integration, 32 browser. Lint clean.

**Not tested:** the hourly job firing by itself on the clock (its selection query was run on real data and the same function is used by "Run now"); Android has no Reports or Automations screen (web only for now).

## 6. Your decisions

1. Should the second person see **"What each person did"**? It is visible to both people now. Say if it should be owner-only.
2. Do you want the Android app to get Reports later, or is web enough for owners?
