# 3. Features

**Status key:** ✅ built and tested · 🔜 built, switched on in a later phase · ⏳ planned, not built

## Features for every business

| Area | Feature | Web | Android | Status |
|---|---|---|---|---|
| Setup | 5-minute setup: business type → profile → import → invite partner → done | ✅ | – | ✅ |
| Setup | Industry templates set names, stages, fields, WhatsApp messages and dashboard | ✅ | ✅ | ✅ |
| Setup | Excel/CSV import with automatic column matching; duplicates skipped | ✅ | – | ✅ |
| Login | Email + password, forgot password (branded email), Google sign-in | ✅ | ✅ | ✅ (needs Firebase keys) |
| Account | 2 people: Owner + Member; invite by email or WhatsApp link; remove member | ✅ | – | ✅ |
| Account | 1 phone + 1 browser per person; owner can log out any device | ✅ | ✅ | ✅ |
| Contacts | Patients / Members / Students with industry fields, tags, search, duplicate check, merge | ✅ | ✅ | ✅ |
| Pipeline | Drag-and-drop stages per industry | ✅ | Move stage | ✅ |
| Appointments | Day and week calendar, staff member, done / no-show / cancel | ✅ | ✅ | ✅ |
| Renewals | Due dates for next visit / membership / fee instalment; mark paid and schedule the next one | ✅ | ✅ | ✅ |
| Tasks | Tasks with reminders; follow-up task created from a call outcome | ✅ | ✅ | ✅ |
| Calls | Click-to-call, then one tap: Interested / Call later / Not interested / No answer | ✅ | ✅ | ✅ |
| WhatsApp | Ready messages filled with name, date, time, amount; opens your own WhatsApp | ✅ | ✅ | ✅ |
| Notes & history | Notes and a full timeline per contact | ✅ | ✅ | ✅ |
| Files | Upload PDF/images/Excel per contact; storage meter | ✅ | Photos (compressed) | ✅ (needs R2 keys) |
| Field visits | GPS check-in saved as a Google Maps link | – | ✅ | ✅ |
| Dashboard | Industry cards + today's appointments, renewals due, tasks | ✅ | ✅ | ✅ |
| Reminders | Morning summary, appointment reminders, renewal alerts, task reminders (push) | – | ✅ | ✅ (needs Firebase) |
| Automatic | No-shows marked at end of day; trial-ending and plan-ending emails | ✅ | – | ✅ |
| Data | Export everything to Excel; activity (audit) log | ✅ | – | ✅ |
| Billing | Manual activation (UPI/bank) with paid-until date | ✅ | – | ✅ |
| Billing | Razorpay subscription and customer payment links | ✅ | – | 🔜 phase 2 |
| Lead capture | Private lead addresses that Google Ads, Facebook (via Zapier/Make), IndiaMART, JustDial or a website form post to; repeats ignored ([30](30-lead-capture-and-scoring.md)) | ✅ | – | ✅ (not yet tested with the live services) |
| Lead scoring | Hot / Warm / Cold with reasons, Leads screen, badge on pipeline cards, Hot leads card | ✅ | – | ✅ |
| Later | WhatsApp Business API inbox, direct Facebook Lead Ads, email-to-lead for property portals, automation rules, Indian-language screens, AI voice notes | – | – | ⏳ |

## What changes per industry

| | Hospital | Gym & Fitness | Education Centre | Retail Shop | Real Estate |
|---|---|---|---|---|---|
| Contacts are called | Patients | Members | Students | Customers | Clients |
| Pipeline | Enquiry → Appointment booked → Visited → Follow-up due → Completed / Not coming | Enquiry → Trial booked → Trial done → Joined → Renewal due / Not joined | Enquiry → Counselling → Demo class → Admission → Enrolled / Not joined | Enquiry → Interested → First purchase → Regular customer → VIP / Not interested | New enquiry → Requirement taken → Site visit → Negotiation → Token / advance paid → Deal closed / Not interested |
| Extra fields | Age, gender, doctor, department, visit reason | Plan, trainer, goal, start date | Course, batch, parent's phone, total fee | Birthday, wedding anniversary, gender, area, likes, customer type | Looking to (buy / rent / sell / let out), property type, BHK, budget, preferred locations, move-by date, home loan needed |
| Appointment types | Consultation, follow-up visit, procedure, health check-up, lab test | Trial session, personal training | Counselling, demo class | Store visit, home delivery, trial / fitting | Site visit, office meeting, documentation / registration |
| Renewal type | Next visit | Membership | Fee instalment | Repeat purchase, service / warranty due, balance payment | Rent agreement renewal, payment instalment, follow-up |
| WhatsApp messages | Appointment confirmation, reminder, follow-up reminder, missed appointment | Trial reminder, welcome, renewal reminder, we miss you | Demo reminder, admission welcome, fee reminder, enquiry follow-up | Thank you, birthday offer, anniversary offer, new arrivals, we miss you, balance reminder | Site visit confirmation, after the visit, new property for you, agreement renewal, payment reminder |
| Dashboard | Appointments today, follow-ups due, no-shows, tasks | New members, renewals this week, overdue, sessions today | New enquiries, admissions this month, fees pending, sessions today | Sales this month (₹), new customers, birthdays this week, not back in 60 days | New clients, site visits today, properties available, follow-ups this week |
| Extra screens | – | – | – | Purchases on each customer | Properties (with matching clients), deals with commission |

All industries also get **Offers**: a WhatsApp message to a chosen group (birthdays, win-back, a tag, clients matching a property),
sent one tap per person from the business's own phone, only to customers who agreed to get offers. Details in
[26 — Retail, real estate and offers](26-industries-retail-real-estate.md).

The templates live in code: `vbw-crm/packages/core/src/templates.ts`. Adding an industry = adding a template.

## Health data rule

Store only contact and appointment details for hospitals — **no medical records**. Health data is sensitive under
India's DPDP Act; get legal advice before storing more.
