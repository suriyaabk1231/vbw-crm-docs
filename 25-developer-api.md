# 25 · Developer API: AI calling, AI booking and other software (built 28 Oct 2026)

A business can now connect its own software to its VBW CRM account. The main use is an **AI receptionist**: an AI voice
agent or a WhatsApp/website chatbot that answers customers, recognises them, books appointments and writes the call
summary into VBW CRM. Everything was tested on the local copy (results in section 7).

**Cost to us: ₹0 extra.** No paid service was added. The business pays its own AI provider (voice agent, chatbot);
VBW CRM only answers its requests. The API runs on the same server.

---

## 1 · How a business turns it on (owner only)

VBW CRM → **Settings → Developer API**:

1. **Create key**: give it a name (e.g. "AI receptionist") and choose **Read and make changes** or **Only read**.
   The key (`vbw_live_…`) is shown **once**, with a Copy button. Give it to whoever sets up the AI agent.
2. Optional: **Webhook**. Type the agent's `https://` address and tick the events to send. The secret (`whsec_…`) is
   shown once. **Send test** checks it works; the screen shows the last result (e.g. "200 OK").
3. The screen also shows the **API address**, the **OpenAPI file** (most AI agent builders can import it to learn
   every action) and two ready-made examples.

The member (second person) cannot see or create keys.

## 2 · What the AI agent can do

Base address: `https://api.<domain>/api/v1/ext`. Every request sends the header `Authorization: Bearer vbw_live_…`.

| Need | Request |
|---|---|
| Check the key works | `GET /me` |
| Business name, services (appointment types), stages, lead sources, call outcomes | `GET /business` |
| **Recognise a caller** by phone (any format: `+91 98400 12345`, `9840012345`) | `GET /contacts?phone=…` |
| Search customers by text | `GET /contacts?q=…` |
| Full customer: history, appointments, follow-ups, open tasks | `GET /contacts/{id}` |
| **Add a lead** (returns the existing customer if the phone is already there) | `POST /contacts` |
| Change name, phone, tags, fields or pipeline stage | `PATCH /contacts/{id}` |
| **Save a call**: outcome + summary + optional call-back reminder | `POST /contacts/{id}/calls` |
| Add a note | `POST /contacts/{id}/notes` |
| **Busy times on a day** (everything else is free) | `GET /availability?date=YYYY-MM-DD` |
| **Book** (by customer id, or by phone; with a name, a new caller is added) | `POST /appointments` |
| **Move, confirm, cancel**, mark done / no-show | `PATCH /appointments/{id}` |
| Appointments between two times | `GET /appointments?from=…&to=…` |
| Follow-ups / renewals / fees due | `GET /followups?view=week` |
| Task for the team ("Call back about the price") | `POST /tasks` |
| The full description for AI tools (no key needed) | `GET /openapi.json` |

Safety rules built in:

- A booking that overlaps another one is refused with **409 SLOT_TAKEN**, so the agent offers another time.
  Bookings in the past are refused.
- `status: "requested"` lets the agent ask for a time that **staff confirm** in the app (shows in the pop-up reminder).
- Everything the API does shows in the customer history and the activity log as **"API: <key name>"**, e.g.
  "Consultation moved to 1 Nov 2026 12:00 pm · API: AI receptionist".

## 3 · Webhooks: VBW CRM tells the agent straight away

| Event | When |
|---|---|
| `lead.created` | A lead is added: in the app, by the API, or from the booking page |
| `enquiry.received` | Someone sends the booking page form |
| `appointment.created` | An appointment is booked (app, API or booking page) |
| `appointment.updated` | An appointment is moved, confirmed, cancelled, done or no-show |
| `test` | The owner pressed **Send test** |

Example use: a Facebook ad lead arrives → `lead.created` → the AI agent calls the person within a minute → it books a
visit with `POST /appointments` and saves the summary with `POST /contacts/{id}/calls`.

Each message is a JSON `POST` with headers `X-VBW-Event`, `X-VBW-Delivery` (unique id), `X-VBW-Timestamp` and
`X-VBW-Signature`. To check it really came from VBW CRM:

```
X-VBW-Signature = "sha256=" + hex( HMAC-SHA256( secret, X-VBW-Timestamp + "." + raw body ) )
```

If the agent does not answer with 2xx within 8 seconds, we keep trying, **6 tries in all**, waiting longer each time (starting
at 30 seconds). After **50 failures in a row** the webhook switches itself off and the screen says so; saving it again
switches it back on.

## 4 · Example: AI receptionist answering a call

1. Call comes in from `+91 98400 12345` → agent calls `GET /contacts?phone=+919840012345`.
   Found → "Hello Lakshmi, welcome back." Not found → it is a new caller.
2. Caller wants a cleaning on Monday → `GET /business` (services) and `GET /availability?date=2026-11-02`.
3. Agent offers a free time → `POST /appointments`
   `{"phone":"9840012345","name":"Lakshmi","typeKey":"consultation","startsAt":"2026-11-02T11:00:00+05:30"}`.
   409 SLOT_TAKEN → offer the next free time.
4. After the call → `POST /contacts/{id}/calls` `{"outcome":"interested","summary":"Booked cleaning Monday 11 am"}`.
5. Staff see the booking in Appointments and the summary in the customer history, marked "API: AI receptionist".

Most AI voice-agent and chatbot builders can call outside APIs ("custom tools", "functions" or "HTTP actions"), and
some can import an OpenAPI file directly. Check the chosen provider's own documentation for how; we have not tested any
specific provider yet.

## 5 · Security

| Risk | What stops it |
|---|---|
| Key stolen from our database | Only a SHA-256 hash and the first 13 characters are stored; the full key cannot be read back |
| One business reading another's data | The key belongs to one business; the database rule (RLS) blocks other businesses. Tested: the gym's key gets "not found" for clinic data |
| A leaked key | Owner presses **Switch off**; it stops at once (tested: 401). Separate keys per tool. At most 10 keys |
| A read-only tool making changes | "Only read" keys get 403 on any change (tested) |
| Too many requests | 300 a minute per key, then 429 |
| Plan ended | Reads still work; changes get 402 until renewed (tested) |
| Webhook sent to our own servers or a private network (SSRF) | Only `https://`; private, local and cloud-metadata addresses are refused when saving **and** checked again at sending time (tested: 192.168.x, 169.254.169.254, plain http refused) |
| Fake webhook messages | HMAC signature with the business's secret; **New secret** replaces it at once (tested: wrong secret detected) |
| Who did what | Key creation, switch-off, webhook changes and secret changes are in the activity log; API actions show the key name |

## 6 · Where it is in the code (`vbw-crm`)

| Part | File |
|---|---|
| Tables `api_keys`, `webhooks` with RLS | `apps/api/src/db/migrations/009_developer_api.sql` |
| Key login, read-only check, per-key rate limit | `apps/api/src/common/auth.ts` |
| Events, webhook signing, SSRF guard | `apps/api/src/common/events.ts` |
| Webhook sending and retries (worker) | `apps/api/src/jobs/jobs.service.ts` (`webhook-deliver`) |
| Owner screens' API and the `/ext` API | `apps/api/src/modules/developer.module.ts` |
| OpenAPI description | `apps/api/src/modules/developer.openapi.ts` |
| Settings → Developer API screen | `apps/web/src/app/app/settings/DeveloperTab.tsx` |

**On deploy:** run the migration (`npm run migrate`) and restart **both** the API and the worker; webhooks are sent
by the worker.

## 7 · Test results (local, 28 Oct 2026)

| Check | Result |
|---|---|
| Owner creates keys; member cannot (403); full key not stored | ✅ |
| No key / wrong key / switched-off key → 401 | ✅ |
| Caller lookup in another phone format | ✅ |
| Add lead; same phone again returns the existing customer | ✅ |
| Book; same time again → 409; in the past → 400; new caller by phone + name | ✅ |
| Move, cancel, call with call-back, note, stage change, task | ✅ |
| History shows "API: AI receptionist", including "moved to …" | ✅ |
| Other business's key on clinic data → 404 / empty | ✅ |
| Plan ended: reads 200, changes 402 | ✅ |
| Webhook: plain http, private IP, metadata IP, unknown event refused | ✅ |
| Events from the API, the app and the booking page all delivered, signatures valid | ✅ |
| Receiver failing → retried after 30 s, failures counted, reset after success | ✅ |
| Settings screen on desktop (dark) and phone (light): create key, shown once, switch off, send test | ✅ |

Not yet done: a test with a real AI voice-agent provider, and the Android app screen (the web screen works on phones).
