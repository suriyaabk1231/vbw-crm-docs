# 30 · Lead capture and lead scoring (built 29 Oct 2026)

Two gaps from the competitor check (a teammate's "CRM basics" list) were closed: **lead capture** (leads arriving by themselves from ads and
portals) and **lead scoring** (Hot / Warm / Cold). Everything here was built and tested on the local copy; what was **not** tested with the
real outside services is listed in section 4.

---

## 1 · Lead scoring: Hot / Warm / Cold

Every customer who is still being won (not in a Won or Lost stage) gets a score from 0 to 100 and a label. **60 and above is Hot, 30 to 59 Warm,
below 30 Cold.** Every point is explained on screen, so staff can see why. The rules are one tested function (`scoreLead` in `packages/core/src/leads.ts`).

| Signal | Points |
|---|---|
| Has a booking, or asked for one (booked or requested visit not yet happened) | +25 |
| Has visited before | +10 |
| Last logged call: interested / asked to call later / not interested | +20 / +5 / −40 |
| Did not answer 2 or more calls in the last 14 days | −10 |
| Moved along the pipeline (0 at the first open stage, up to +20 at the last open stage) | 0 to +20 |
| Anything logged in the last 3 days / 2 weeks | +15 / +8 |
| Nothing logged for over 30 days / over 60 days | −15 / −25 |
| New enquiry (added in the last 2 days): reply fast | +10 |
| Phone number known | +5 |
| Source: referral 15, walk-in 10, phone 10, WhatsApp 8, website 8, QR 8, Google 8, property portal 6, IndiaMART 6, Facebook/Instagram 4, JustDial 4, email 4 | +4 to +15 |
| Real estate: each requirement filled in (looking to, type, BHK, budget, locations, move-by) | +5 each, up to +15 |
| Real estate: an available property fits (same rule as the property list) | +15 |

**These weights are my starting guesses, not measured.** The source points in particular should be compared with which sources actually become
customers after the pilot; change `SOURCE_POINTS`, `HOT_FROM` and `WARM_FROM` in `leads.ts` and the unit tests will show what moved.
Won, lost, deleted and merged customers are never scored. Scoring is calculated when you look (not stored), so it is always current.

**Where it shows**
- **Leads** (new menu entry): Hot / Warm / Cold with counts, best first, the top reasons, a Call button. The page shows the best 300 (the API can return up to 500); scoring covers up to 5,000 open customers per business.
- **Customer page:** a "Lead score" card listing every reason with its points.
- **Pipeline:** a Hot / Warm / Cold badge on each card.
- **Home:** a **Hot leads** card for real estate, education centres and gyms (their templates went to version 2).
- **Developer API:** `GET /ext/contacts/{id}` also returns `leadScore`.

## 2 · Lead capture: lead addresses

In **Settings → Lead sources** (owner only) the owner makes an **address**: a private web address such as
`https://api.<domain>/api/v1/public/leads/vbl_…`. Another service posts new leads to it. Each address has a name and a lead source, and each
lead that arrives:

1. is read, whatever the sender calls its fields (name, phone, email, message, city, company, interest);
2. becomes a **customer** in the first open stage, marked with the address's source; a known phone number or email is **added to that customer**
   instead of copied (missing details are filled in, nothing the business already has is overwritten);
3. gets a history entry "Lead from <address name>" with the message and any extra answers, shown as "Lead: <address name>";
4. gets a **"Call back" task** for the owner (not a second one if one is already open);
5. triggers the same alerts as the booking page (the pop-up on opening the app, the home screen) and the `lead.created` and
   `enquiry.received` webhooks of the Developer API;
6. for real estate, saves the buyer's answers (looking to buy/rent, property type, BHK, budget such as "85 lakh" or "1.2 crore", locations) as fields,
   so matching and scoring work at once.

Offers consent is **not** set by a lead (a lead form gives permission to be contacted, not to get offers). It is still a separate tick.

**What it accepts:** POST with JSON or a web form, and GET with query parameters (JustDial-style pushes). It recognises the **Google Ads lead form**
shape (`user_column_data` with `column_id` and `string_value`), **Meta Lead Ads** answers (`field_data` with `name` and `values`), an **IndiaMART-style** push, and plain forms.
Answers it does not recognise are kept in the history text, not dropped. A repeat delivery of the same lead id is ignored. A lead with no phone and no
email is refused (422) and shown, with what the sender posted, under "See what arrived".

**From the owner's screen:** a shown-once address with a Copy button, a list of addresses with last lead and count, **Send a test lead** (goes through
exactly the same steps and tags the customer "test"), **See what arrived** (the last 50 deliveries and why any failed), Switch off and Delete.
At most 10 addresses per business. Step-by-step guides for each service are on the same screen, each with an honest status line.

## 3 · Security

| Risk | What stops it |
|---|---|
| Guessing an address | 24 random bytes (192 bits); only a SHA-256 hash is stored (like an API key); the address is shown once. "No such address" and "switched off" give the same 404 |
| Someone posting junk | The secret is the key; 120 requests a minute per sender address; body limit 1 MB; reading is capped (depth, number of fields, field length); a lead cannot name another business |
| A lead landing in the wrong business | The address decides the business; a `business_id` in the lead is ignored (tested) |
| Another business reading the log or deleting the address | 404 (tested); the two new tables have the same row-level security and same-business checks as every other |
| A website form being blocked by the browser | Only the lead-address path allows any website to post (no login, no cookies, nothing readable); the rest of the API still allows only your own app (tested) |
| Personal data kept in the log | The log keeps the outcome and a link to the customer; the raw lead is kept only for a lead that could not be read (cut to 4,000 characters), and only the last 50 per address |
| Plan ended | New leads are refused with 402 until renewed (the sender will see the error and retry) |

## 4 · What was tested, and what was not

| Source | Status |
|---|---|
| Plain website form, form post, GET with query parameters | ✅ tested here, including from another website's origin |
| Google Ads lead form | Read from Google's documented shape (`lead_id`, `user_column_data`, `google_key`). **Not tested with a live Google Ads account** ([Google's webhook documentation](https://developers.google.com/google-ads/webhook/docs/implementation)). We ignore `google_key`; the secret in the address is the key |
| Facebook / Instagram lead ads | Meta's own webhook sends only a lead id; the details must then be fetched with an approved Meta app and the `leads_retrieval` permission ([Meta: retrieving leads](https://developers.facebook.com/docs/marketing-api/guides/lead-ads/retrieving)). **Not built.** What is built: leads passed on by Zapier, Make or Pabbly in the documented `field_data` shape. **Not tested with a live Facebook account** |
| IndiaMART | IndiaMART's help describes a Push API that posts JSON to a webhook URL ([IndiaMART help](https://help.indiamart.com/knowledge-base/integration-of-indiamarts-lead-manager-crm-push-api-with-third-party-crms-real-time-push-of-leads)). **I could not confirm its field names**, so the reader accepts common ones (`SENDER_NAME`, `SENDER_MOBILE`, `QUERY_MESSAGE`, `UNIQUE_QUERY_ID`... also inside a wrapper such as `RESPONSE`). The first real lead may need a one-line fix; it will show under "See what arrived" with what IndiaMART sent. Our reply includes `CODE: 200, STATUS: "SUCCESS"`, which is a guess at what it wants |
| JustDial | Other CRMs report that JustDial's portal has no field for a web address; their account manager sets it, with GET ([LeadSquared's guide](https://help.leadsquared.com/integrate-justdial-with-leadsquared/)). Accepted as GET or POST. **Not tested; parameter names unconfirmed** |
| 99acres, MagicBricks, Housing | **Not built.** I found no public lead API for CRMs; they email leads. Reading emails needs an inbound-mail service (not chosen, cost unchecked) |

Automated tests: 41 in `packages/core` (scoring rules, every payload shape, budgets, hostile input) and 27 integration tests in
`apps/api/test/integration/leads.test.ts` (addresses, every format, repeats, bad and switched-off addresses, plan ended, isolation, log limit, test lead, browser
pre-check, real estate fields, all score cases, ties, Home card). Total now: 60 unit and 108 integration tests, all passing, lint clean, build clean.
Checked in the browser too: making an address, posting leads, "See what arrived", the Leads page, a customer's score card and the pipeline badges, at desktop and phone width.

Two real bugs were found by testing and fixed: budgets such as "₹80,00,000" were read as 80 (commas), and the lead list failed when two leads had the same score
(dates from the database were not text).

## 5 · Not done, and next steps

1. **First live check of each connector** (Google Ads, Facebook through Zapier or Make, IndiaMART) with the owner's own accounts; fix field names from the first real lead.
2. **Direct Facebook / Instagram Lead Ads** (needs a Meta app, business verification and review) and **email parsing** for 99acres, MagicBricks and Housing (needs an inbound-mail provider).
3. **Tune the score** after the pilot (weights per industry, "no answer" rules, source points) using which leads actually converted.
4. **(Done 30 Oct, see [33](33-reports-and-automations.md); deal value still open)** **Automatic tasks** when a lead stalls or changes stage ("no activity for 7 days") and a simple **deal value** on pipeline cards: the other two gaps from the competitor check.
5. The Android app does not show the Leads screen yet (the web app on the phone does).
