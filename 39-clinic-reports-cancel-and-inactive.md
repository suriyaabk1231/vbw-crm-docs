# 39 — Clinic reports, cancel flow and inactive patients: what was built (2 Nov 2026)

*Built from the checklist the owner pasted on 2 Nov 2026 (billing and inventory left out on purpose). "Built" means written, type-checked, linted and unit-tested; the screens were clicked through against a stand-in API. **The new database queries have not been run against a real database** (Docker would not start on this PC): run `npm run migrate` and `npm run test:integration` on a machine with Docker before trusting them.*

## 1. The idea: no dead ends

Every screen leads to the next step, so a patient never falls between two features.

```
Lead (source) -> Booked -> Came (Done) -> Follow-up due -> Due soon list
                    |                           |
                    +-> Cancel / No-show -> reason -> [Book again | WhatsApp | Call-back task]
                                                   -> "Cancelled, not booked again" (Reports)
Stopped coming (missed follow-up / no visit) -> Inactive screen -> [Call | WhatsApp | Book]
```

## 2. What the owner can do now

| Where | What |
|---|---|
| Calendar and Home | **Cancel** and **No-show** ask "Why?" (Feeling better, Cost, Time clash, Going elsewhere, We had to cancel, Other: optional). Then: **Book a new appointment**, **Message on WhatsApp**, or **Remind me to call tomorrow** (adds a task for 10:00). The reason also shows on the card and in the patient's history. Home now has a Cancel button too. |
| Reports | Custom dates ("Choose dates") plus 7 / 30 / 90 / 365 days. Lead sources now show **Added, Booked, Came, Won**. New cards: appointments (done, cancelled, no-show, coming up, follow-up visits, cancelled-or-missed %), **why they cancelled**, **new and returning**, **by doctor**, and **Cancelled or missed and not booked again** with Call and WhatsApp. A notice warns when past appointments were never marked Done, No-show or Cancelled. Excel has all of it. |
| Inactive (new menu entry) | Lists people who stopped coming, longest first, with Call, WhatsApp and Book. The owner can change what "inactive" means. |

## 3. Rules (one place: `packages/core/src/patients.ts`, tested)

- **Inactive** = a follow-up has been missed for at least **15 days** (default), **or** no completed visit for **90 days** (default), **and** nothing is booked ahead. People who never had a visit are leads, not inactive. Both numbers are per business (1 to 730 days). *The defaults are a starting point, not a standard: published guidance varies from 180 days in one follow-up study to 12–36 months in primary care.*
- **Returning** = completed a visit in the period and had a completed visit before the period. **New** = the first completed visit is inside the period. (Simple on purpose; the US 3-year billing rule does not fit a small clinic.)
- **Period** = Indian dates, both ends included; a future end date is cut back to today; at most 800 days.
- Cancel reasons are our own list, not from research (no solid source on why patients cancel was found).

## 4. Where it is

| Part | Where |
|---|---|
| Shared rules and 16 new tests | `packages/core/src/patients.ts`, `patients.test.ts` |
| Database (migration 013) | `appointments.status_reason`, `status_changed_at` |
| API | `POST /appointments/:id/status` accepts `reason`; `GET /reports?days=` or `?from=&to=`; `GET /inactive`; `PATCH /settings/business` accepts `inactiveRule`; `/me` returns it |
| Web | `components/app/outcome.tsx`, `app/app/reports/page.tsx`, `app/app/inactive/page.tsx`, menu in `Shell.tsx`, `calendar/page.tsx`, `app/page.tsx` |

## 5. Not done

- Android app: unchanged (still works; no cancel-reason sheet, Inactive or new reports yet).
- Waiting list (offer a cancelled slot to a waiting patient), PDF export, branch and doctor filters, billing, OPD queue, inventory: not built.
- No browser test or integration test was added for the new screens; `/app/inactive` is not yet in the accessibility scan.
- "Doctor" means the person an appointment is assigned to (a login), not a separate doctor list.
