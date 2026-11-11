# 24 · Online booking, leads and the pop-up reminder (built 27 Oct 2026)

What was added so customers can book from anywhere, every lead lands in the CRM tagged with where it came from, and
staff see what needs doing the moment they open the app. All tested on the local copy.

---

## 1 · The booking page (one per business)

- Address: `https://app.<domain>/b/<business>`, e.g. `/b/sri-sai-hospital-7f3a`. Made automatically at sign-up from the
  business name; the owner can change it (Settings → **Online booking**). Existing businesses got one too.
- No app, no login. The customer types name, phone (WhatsApp), optional email, picks **"Just an enquiry"** or a
  service (Consultation, Trial session, Demo class… from the industry template), a preferred date and time, a message,
  and ticks consent ("I agree that <business> may contact me…").
- After sending: a thank-you screen with **Call** and **WhatsApp** buttons for the business.
- Works on phones and computers, light and dark, and can sit **inside the business's own website** (`?embed=1`).
- Name tag on every page: **"Online booking by VBW CRM"**, linking to our website, so every client's customers see us.
- The owner can **pause** it (the page then asks people to call) and add a **note** (hours, address).

## 2 · What happens with each enquiry

1. A new lead is created in the first pipeline stage, tagged `online` and with its **source**. If the phone number is
   already a customer, the enquiry is added to that person instead (no duplicate; the page never reveals who is a customer).
2. History entry: "Online enquiry from Facebook / Instagram: <message>".
3. If a time was chosen: an appointment marked **Request** (amber). Staff press **Confirm** (becomes Booked) or
   **Decline** on Home or Appointments; history says "Booking confirmed" / "Booking request declined".
4. A task for the owner: "Call back <name>" or "Confirm the booking request from <name>".
5. It pops up in the app (section 4).

## 3 · Top 5 places to get leads (all in Settings → Online booking, with copy buttons)

| # | Place | What the business does | Leads are tagged |
|---|---|---|---|
| 1 | **Ads** (Facebook, Instagram, Google Ads) | Paste the link as the ad's website link and in the Instagram bio | Facebook / Instagram, Google |
| 2 | **WhatsApp** | Paste the ready message as the WhatsApp Business **greeting message**; share the link in chats and status | WhatsApp |
| 3 | **QR code** | Download it; print on posters, flyers, visiting cards, the reception desk | QR code / poster |
| 4 | **Their own website** | Paste the booking form code (iframe) or the "Book now" button code | Website |
| 5 | **Google Business Profile** (Google Maps) | Add the link as the Appointment / Booking link | Google |

Each link carries `?src=…`; anything unknown counts as Website. Staff can also set the source by hand on any
customer (Walk-in, Referral, Phone call, Other). The Home screen shows **"New <customers> this month, by where they came
from"** as bars, so the owner sees which ads work.

## 4 · The pop-up reminder

When someone opens the app (once per browser session), a pop-up shows:
- **New enquiries** from the booking page since they last pressed "Got it" (name, source, time)
- **Booking requests** waiting to be confirmed, with a button to the Appointments page
- **Overdue tasks** (count and the 5 oldest)
- Shortcuts: appointments today, follow-ups due, tasks

While the app stays open it checks every **2 minutes**; a new enquiry opens the pop-up again. "Got it" marks
everything up to now as seen, for that person only. (The Android app, when published, also gets phone notifications.)

## 5 · Protection

| Risk | Protection |
|---|---|
| Bots filling the form | Hidden trap field; forms sent in under 3 seconds refused; consent required |
| Flooding | 6 enquiries per 10 minutes per internet address; 200 per business per day |
| Seeing other customers | The page shows only the business's public details; the reply is the same for new and known numbers |
| Wrong business | Each enquiry is written inside that business only (row-level security) |
| Embedding | Only `/b/` pages may be shown inside other websites; every other page refuses |
| Paused or unpaid account | The page stops taking bookings and shows the business phone number |

## 6 · Tests (27 Oct 2026, local copy)
Booking page on phone and desktop, light and dark; enquiry; booking request; known phone number; unknown source tag;
bot field; too-fast form; no consent; past time; bad phone; 7th request in 10 minutes refused; paused page; member
cannot change settings; page address already taken refused; pop-up content and greeting; pop-up shown once per session;
confirm request; dashboard source bars; source picker saves and edits; settings tab and QR code; no sideways scrolling
on phones; `/b/` embeddable while `/app` is not. All 30 automatic tests pass; web, API and mobile type-check; full build passes.

Also fixed while testing: clicking Next/Back quickly on Appointments (or switching tabs on Follow-ups and Tasks) could
show the previous day's list under the new date; the latest request now always wins.

---

## 7 · Key features still missing (review, 27 Oct 2026)

| Priority | Missing | Why it matters | Effort (*estimate*) |
|---|---|---|---|
| **High** | **Payment receipts** (PDF / WhatsApp) and a **monthly collections report** | Clinics and gyms give receipts; owners want "how much came in this month" | 1–2 days |
| **High** | **Monthly report**: new leads, conversions, visits done, no-show rate, money collected | Owners judge the CRM by this; good for renewals of our own plan | 1–2 days |
| **High** | **Google review request** after a visit is marked Done (WhatsApp template with their Google review link) | Cheap, visible value; helps their Google ranking, which brings more booking-page leads | ½ day |
| Medium | **Opening hours and slots** on the booking page (only times they are open) | Fewer requests at impossible times | 1 day |
| Medium | **Birthday / special-day greetings** (date field + WhatsApp template + daily list) | Popular in Indian businesses; keeps customers coming back | ½–1 day |
| Medium | **Tamil (and later Hindi) screens** | Many target owners and staff prefer Tamil; strong differentiator | 3–5 days |
| Medium | **Two-step login** for owners | Security; Firebase needs its paid Identity Platform upgrade (check the price first) | 1 day + cost |
| Later | Automatic WhatsApp sending, WhatsApp inbox, Facebook lead-form import | Needs Meta approval and per-message charges (phase 4) | weeks |
| Later | Android app restyle, real-phone tests and Play Store (phase 3) | Phone notifications for staff | 1–2 weeks |

## 8 · Security note found during this work
`npm audit` reports **23 known issues in existing packages** (9 high, 14 moderate, none critical), inside Next.js, NestJS,
Firebase, zod and others. One safe fix was applied (`node-forge`). The rest need **major version upgrades**
(Next.js 16, NestJS 12, newer Firebase and zod), to be done as a separate, fully re-tested task before launch.
