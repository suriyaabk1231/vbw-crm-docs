# 36 — Design: plan lengths, calendar dates and a "Due soon" reminders screen

*Design document, 31 Oct 2026. **Status: approved and built the same day, see [38](38-due-soon-reminders-built.md).** Sections marked "my call" are choices I made, not things the owner agreed to.*

## 1. Intent (the why)

Owners should not have to search for who is about to lapse or who is due back. When a person is added, or renewed, the owner picks **how long** (for example 3 months for a gym, 12 days for a dental follow-up) **or picks a date on a calendar** (for example 16 December). The app then reminds the owner **before** it ends, shows everyone who needs action on **one separate screen**, and lets them **call, WhatsApp, renew or upgrade** from there. Each industry gets its own lengths and its own words.

Research (31 Oct 2026, mostly vendor sources, so timing patterns only) agrees on: several reminders before the date (7, 3 and 1 day for gyms; 90, 60 and 30 for rent agreements), one follow-up after it lapses, a per-patient interval for dental recalls, and WhatsApp as the channel in India.

## 2. Goals, non-goals, anti-goals

**Goals**
1. Choose **"after N days / months"** or **"pick a date"** when creating a follow-up or renewal, and when renewing.
2. **Several reminder points** per item (default per industry; the owner can change them), plus an optional one **after** the date passes.
3. A **"Due soon" screen** (web and Android) listing people who need action now, with Call, WhatsApp, Renew / Upgrade, Snooze, Not renewing.
4. A **push to the owner's phone** when a reminder point is reached, naming the person.
5. **Upgrade** at renewal: pick a longer length in one tap (gym: up to 18 months).
6. A short **note** on every item (it already exists) shown on the reminder card, for example "bring the X-ray".

**Non-goals (things this will not do)**
- It does **not** send messages to customers by itself. WhatsApp stays a tap-to-send link (the WhatsApp Business API is not built).
- It does **not** change payments, prices, the Developer API, the Excel export, the dashboard cards, or any other screen.
- No separate "plans" or billing-history system (approach B was rejected).
- No start-date field and no owner-editable list of plan lengths in this version (lengths come from the industry template; any date can still be picked).

**Anti-goals (failure even if it "works")**
- Existing renewals, their screen, views and the 9 AM summary push must behave **exactly as today**.
- A reminder must never fire twice for the same point, and never fire for a person who already renewed, cancelled, was deleted or is snoozed.
- A view-only (lapsed) account must not get new reminders created.

## 3. What exists today (checked in the code)

- `renewals` table: type, due date, amount, note, status `due / paid / renewed / cancelled`.
- Renewals screen (web and Android) with Week, Overdue, Upcoming views; add form (`FollowUpDialog`); "mark paid / renew" row (`MarkPaidRow`) that creates the **next** renewal copying only type and amount.
- `renewalAlerts` job, 09:00: one summary push with counts. WhatsApp sheet already takes a `renewalId`.
- Industry templates already define renewal types (gym "Membership", hospital "Next visit", education "Fee instalment", retail 3 types, real estate 3 types).
- The renewals label differs by industry ("Renewals", "Follow-ups", "Fees", "Reminders").

## 4. Approaches considered

| | What | Verdict |
|---|---|---|
| **A. Extend renewals** | Add a few columns, plan options in templates, a derived "Due soon" screen | **Chosen by the owner.** Reuses everything that works |
| B. New plans system | Separate plan, start, end and upgrade-history tables | Rejected: bigger, duplicates renewals |
| C. Form change only | Days-or-date choice and reminder days, no new screen | Rejected: the owner asked for a separate screen |

## 5. The design

### 5.1 Data (migration 012, additive only)

New columns on `renewals` (all with safe defaults, so existing rows work unchanged):

| Column | Meaning |
|---|---|
| `plan_length` int, `plan_unit` `'days'` or `'months'` (both null for a date picked by hand) | For display ("3 months") and so a renewal can repeat the same length |
| `remind_days` int[] default `{}` | Reminder points. A positive number = that many days **before** the date; a negative number = days **after** (`-1` = one day after). Empty = no automatic reminders (every existing row) |
| `reminded_days` int[] default `{}` | Points already sent, so none fires twice |
| `snoozed_until` date null | Hidden from "Due soon" until that date |

Rules (a pure, tested function in `packages/core`): a point is **due** when `today >= due_date - point`; a point earlier than the day the item was **created** is skipped (so a 1-month plan never gets a "30 days before"); if several points were missed (server down), **only the latest missed one fires**.

### 5.2 Dates (my call)
When the owner chooses "after N days/months", the **server** works out the date in Indian time (Asia/Kolkata), not the phone or browser. Months follow the calendar and clamp to the month end (31 Jan + 1 month = 28 Feb). A picked calendar date is used as given; it may not be in the past for a new item.

### 5.3 Per-industry defaults (in `templates.ts`; my call, owner can edit reminder points per item)

| Industry | Type | Lengths offered | Default reminder points |
|---|---|---|---|
| **Gym** | Membership | **1 to 18 months** (every month) or a date | 7, 3, 1 days before, 1 day after. 5 and 30 selectable |
| **Hospital (dental and others)** | Next visit / follow-up | **3, 5, 7, 12, 18, 30 days** or a date | **3 days before** (as the owner asked) |
| **Education** | Fee instalment | 1, 2, 3 months or a date | 3 days before, 1 day after |
| **Retail** | Repeat purchase | 7, 15, 30, 60, 90 days or a date | 3 days before |
| | Service / warranty due | A date (from the product) | 7, 3 days before |
| | Balance payment | A date | 3 days before, 1 day after |
| **Real estate** | Rent agreement renewal | **11 months** (existing cycle) or a date | **90, 60, 30** days before (edit to match the deed's notice period) |
| | Follow-up | 2, 5, 7 days or a date | 1 day before |
| | Payment instalment | A date | 3 days before |

Selectable reminder points in the form: 1, 2, 3, 5, 7, 15, 30, 60, 90 days before, plus "1 day after".

### 5.4 API

| Change | Detail |
|---|---|
| `POST /renewals` (modify) | Accepts **either** `dueDate` **or** `after: {length, unit}`; optional `remindDays` (up to 6 whole numbers between -30 and 365, no repeats). Without the new fields it behaves as today |
| `PATCH /renewals/:id` (modify) | Same optional fields |
| `POST /renewals/:id/status` (modify) | When renewing, also accepts `nextAfter` (an upgrade) and **copies the plan length and reminder points to the next item** (today it copies only type and amount) |
| `GET /due-soon` (new) | People who need action now: status `due`, a reminder point reached, not snoozed; overdue ones included; sorted by date; includes note, plan text, and a count |
| `POST /renewals/:id/snooze` (new) | `{until}`; hides it until that day |
| Everything else | Unchanged, including `GET /renewals` and its views |

Both people can use all of it (same as renewals today). A view-only account can read but not change.

### 5.5 The reminder push (job `plan-reminders`, new, 09:05)
For every paid-up business, find items whose reminder point is due and not yet sent, mark the point as sent, and push to the owner and the second person ("Asha: Membership ends in 3 days"). More than 5 in one run are grouped ("Asha and 6 more are due soon"). The existing 09:00 summary job is **left as it is**.

### 5.6 Screens

- **Add follow-up form (web and Android):** "When" has two choices: **After** (length chips from the industry list) or **Pick a date** (calendar). The resulting date is shown in words ("Wed 16 Dec 2026"). "Remind me" chips (multi-select) and a "1 day after" switch, pre-filled from the industry default. The existing note field stays.
- **Renew / upgrade row:** same choice; the current length is pre-selected and longer ones are marked "Upgrade". Amount stays.
- **"Due soon" screen (new; name chosen because "Alerts" and "Reminders" are already used elsewhere):** web page `/app/due-soon` with a menu entry and a count badge; on Android a bell with a count at the top of Home that opens the screen (no sixth tab, my call). Each card: person, type and plan ("3 months"), the date and "in 3 days / 2 days overdue", the note, and buttons **Call, WhatsApp, Renew / Upgrade, Snooze (1 day, 3 days, pick a date), Not renewing**.
- **Client page:** the renewals list shows the plan ("3 months") next to the date. Nothing else changes.
- Words follow the industry (patient, member, student, customer, client).

### 5.7 Safety and privacy
Same row-level security and same-business checks as renewals; the new endpoint is limited to the signed-in person's business. Reminder text contains only the person's name, the type and the date (already visible to staff); the note is shown in the app, **not** in the push. Plan-ended accounts get no new reminders.

## 6. Testing plan
1. **Unit (core):** date math (days, months, month-end clamp, past-date refusal), reminder-point rules (skip before creation, latest missed only, none twice), template option lists for all five industries.
2. **API integration:** create by length and by date; validation limits; reminders appear at the right day (dates moved in the database, as the other tests do); snooze; renew with upgrade copies plan and points; cancel; both people; other business sees nothing; view-only account; existing renewals tests unchanged.
3. **Browser:** the add form (both choices, the date shown), the Due soon screen and its buttons, upgrade at renewal, plus the Due soon page in the accessibility scans.
4. **Regression:** the existing 145 API and 32 browser tests must still pass.
5. Android: type-checked; **not** run on a phone (same as the other Android work).

## 7. What is left to the builder
Exact wording of buttons and empty states; chip layout; the shape of the Android bell; naming of the migration file; how the job reads "today" in Indian time (reuse the existing `TODAY` pattern).

## 8. Risks and honest limits
- Reminders reach the **owner**, not the customer. Customers get a message only when the owner taps a WhatsApp link.
- Defaults come from vendor blogs (timing patterns), not from measured results. They are editable per item.
- Real-estate notice periods must be checked against each registered lease deed.
- Push uses Firebase and is **not tested for real** (no Firebase account yet).
- Android is type-checked only.

## 9. Calls I made (please check)
1. The screen is called **"Due soon"**.
2. Android gets a **bell with a count on Home**, not a sixth tab.
3. Gym default is **7, 3, 1 days before + 1 day after** (your example was 5 days; 5 is selectable).
4. Dental / hospital default is **3 days before**, as you asked.
5. The **server** computes "after N days/months".
6. Existing 09:00 summary push stays; the new per-person push is a **separate 09:05 job**.
7. Both people see and act on Due soon.
8. No owner-editable length lists and no start date in this version.
