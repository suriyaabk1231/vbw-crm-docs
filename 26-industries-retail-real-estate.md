# 26 · Hospital, Retail Shop, Real Estate and Offers (built 28 Oct 2026)

Owner's request: rename **Clinic** to **Hospital** everywhere, add **Retail** and **Real Estate**, and give each industry
what it actually needs instead of one CRM for everyone. Retail especially needs its own customer data for promotional
offers. Everything below was built and tested on the local copy (section 8).

VBW CRM now has five industries. Choosing one at setup loads its own words, stages, fields, appointment types,
reminders, WhatsApp messages, dashboard cards and screens. The full comparison table is in [03 — Features](03-features.md).

---

## 1 · What the research said (sources)

These come from vendor pages and guides, not independent studies, so they show what the market offers and expects.
Prices and claims were not checked further.

**Real estate CRMs in India** offer: lead capture from property portals, requirement capture, property inventory,
**matching buyers to properties**, site-visit scheduling, WhatsApp follow-up and commission tracking.
- [Sell.Do](https://www.sell.do/real-estate-crm): inventory, property matching, site-visit workflows.
- [LeadSquared](https://www.leadsquared.com/real-estate-crm/): alerts when a site visit is booked, agent tasks.
- [MagicBricks READPRO](https://www.owler.com/reports/magicbricks/magicbricks--magicbricks-enhances-readpro-crm-for-/1737820562909): MagicBricks' own broker CRM, reported 15,000+ brokers.
- Makanify (via [iTechGuides](https://www.itechguides.com/products/makanify/)): broker plan about ₹500 per user a month with portal leads, site visits and commission tracking.

**Retail CRMs** for Indian shops focus on: purchase history, **birthday and anniversary offers**, thank-you messages
after purchase, loyalty, and WhatsApp as the main channel.
- [Zoho RetailIQ](https://www.zoho.com/retailiq/retailcrm/turn-footfall-into-customer-intelligence.html), [Zithara](https://zithara.ai/blogs/23), [Billfree](https://billfree.in/super-crm).
- Win-back: judge "lapsed" against how often a customer usually buys, and target former good customers first
  ([AMP on RFM](https://useamp.com/blog/rfm-analysis/)). We start with simple fixed choices (60 and 90 days).

**Rules that shaped the design:**
- **Consent for offers.** Under the DPDP Act, consent can be withdrawn at any time and withdrawing must be as easy as
  giving it ([section 6](https://dpdpa.com/dpdpa2023/chapter-2/section6.html)). So offers go only to customers marked
  "agrees to get offers", every message says how to stop, and every change is written in the customer's history.
  The lawyer should confirm this (open question in [17](17-terms-privacy-draft.md)).
- **WhatsApp Business broadcast lists** reach only people who saved the business's number, up to 256 per list
  (vendor guides, e.g. [Botsailor](https://www.botsailor.com/blog/whatsapp-broadcasting-list-a-guide-to-whatsapp-bulk-messaging-2024);
  check WhatsApp's Help Center). So VBW CRM sends **one tap per person** with a wa.me link instead: it works without the
  customer saving the number and needs no paid WhatsApp API.
- **Rent agreements run 11 months** in most states to avoid compulsory registration (over 12 months) under the
  Registration Act, 1908; Maharashtra registers all of them ([Housing.com](https://housing.com/news/is-registration-of-an-11-month-rent-agreement-mandatory),
  [Outlook Money](https://www.outlookmoney.com/invest/why-rental-arrangements-in-india-often-last-only-11-months)).
  So a rental deal can create a renewal reminder 10 months later.
- **RERA:** agents must quote their registration number, and the project's, on advertisements
  ([Mondaq](https://www.mondaq.com/india/real-estate/625646/real-estate-agent-registration-|-real-estate-regulation-act-2016)).
  Each property has a RERA number field, included when the property is shared. Rentals are outside RERA.

## 2 · Hospital (was Clinic)

- Same as before plus a **Department** field (General medicine, Paediatrics, Gynaecology, Orthopaedics, Cardiology,
  Dental, ENT, Eye, Skin, Other) and **Health check-up** and **Lab test** appointment types.
- Existing clinic accounts moved automatically (migration `010`). Still **no medical records**: contact,
  appointment and follow-up details only.
- Aimed at small and mid-size hospitals' front desks; large hospitals need a hospital management system.

## 3 · Retail Shop

- Customers with birthday, wedding anniversary, gender, area, likes (brands, sizes, colours) and customer type.
- **Purchases** on each customer: amount, date, what they bought; total spend and last purchase shown on top.
- Home screen: **sales this month (₹)**, new customers, **birthdays this week**, **not back in 60 days**.
- The pop-up on opening the app lists **birthdays and anniversaries today**, to send wishes.
- Reminders for repeat purchase, service / warranty due and balance payment.

## 4 · Real Estate

- Client requirement: looking to buy / rent / sell / let out, property type, BHK, budget, preferred locations
  (comma-separated), move-by date, home loan needed.
- **Properties**: title, sale or rent, type, BHK, price (or monthly rent), area, location, status (available, on hold,
  sold, rented), owner / landlord, project RERA number, notes. In the phone's bottom bar.
- **Matching**: a client fits a property when they want to buy and it is for sale (or rent and rent), and every
  requirement they filled in fits: type, BHK, budget at least the price, and one of their locations appears in the
  property's location. Empty requirements match anything; clients already won or lost are left out.
  Each property shows its matching clients; each client shows matching available properties.
- **Share on WhatsApp** in one tap: title, type, price, area and RERA number, ready to send. Noted in the history.
- **Deals** with commission; can mark the property sold or rented and add the rent-renewal reminder.
- Home screen: new clients, site visits today, properties available, follow-ups this week.

## 5 · Offers (every industry)

1. **Offers → New offer.** Pick who: everyone who agreed, birthdays (this week / month), anniversaries, not back in
   60 or 90 days, bought in the last 30 days, spent ₹10,000+, clients looking to buy / rent, or clients matching a
   property. Narrow by tag, stage or where they came from.
2. The screen counts who will get it and how many fit but **have not agreed** (left out).
3. Write the message ({name} and {business} are filled in) or start from the industry's messages; see a preview.
4. **Send list**: one WhatsApp button per person, a "send to next" button and progress. Each tap is noted, so nobody
   gets it twice. Customers who stop offers in the meantime are skipped. At most 1,000 people per offer.

Consent can be set when adding a customer, ticked by the customer on the booking page (optional second box), or
changed on the customer's page. When someone replies STOP, staff untick it; this is not automatic yet.

## 6 · Developer API additions ([25](25-developer-api.md))

- `GET /ext/properties`: an AI agent can answer "what 2 BHK flats do you have in Adyar under ₹80 lakh?".
- `POST /ext/contacts/{id}/sales`: billing software can record purchases; agents can record deals.

## 7 · Data safety

- New tables `properties`, `sales`, `campaigns`, `campaign_sends` have the same Row-Level Security and
  same-business checks as every other table. Tested: another business gets "not found".
- Deleting a property or purchase writes its details to the activity log. Merging duplicate customers moves
  purchases, offers and property ownership to the kept record.
- Excel export adds Purchases / Deals and Properties sheets (only when there is data).
- The one-business restore script now includes these tables, and the API keys and webhook tables it was missing.

## 8 · Tested (local, 28 Oct 2026)

| Check | Result |
|---|---|
| Five industries in setup; old clinic accounts now Hospital with Department field | ✅ |
| Retail: purchases, totals, dashboard cards, birthday in the pop-up | ✅ |
| Offers: groups counted, no-consent left out, unknown filter refused, nobody-agreed refused, preview, send list, sent noted | ✅ |
| Customer stops offers after the list was made: sending refused | ✅ |
| Real estate: properties, matching both ways, budget / BHK / location respected, share message, deal marks rented, renewal 10 months later | ✅ |
| "Send to all matching" from a property: only matching clients | ✅ |
| Another business: property, deal, offer all "not found" | ✅ |
| Excel export with Deals and Properties sheets | ✅ |
| Phone width (375 px): landing, setup, properties, property, client page, offers: no sideways scroll | ✅ |
| Tests 32 / 32, type-checks (API, web, mobile), production build | ✅ |

Found and fixed while testing: "new listing" offer picked clients outside budget; wrong dashboard word for real
estate follow-ups; budgets shown as 8000000 instead of ₹80,00,000; matching list not refreshing after a deal;
"1 customers"; and **when the server or internet was down, logged-in people were sent to the login page** (now they
see "Cannot reach VBW CRM" with Try again).

## 9 · Not done yet

- **Android app** shows the new industries' words and fields but not Properties, Purchases or Offers (use the web
  app on the phone for those).
- Replies of STOP are not read automatically (needs the paid WhatsApp Business API).
- Property photos, portal (99acres / MagicBricks) lead import, loyalty points, and a per-customer "usual gap
  between purchases" for win-back.
- Lawyer to confirm the offers consent wording and the updated privacy policy field list.
