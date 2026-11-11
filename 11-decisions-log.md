# 11. Decisions log

Every important decision, newest first. Add new decisions at the top.

| Date | Decision | Reason |
|---|---|---|
| 29 Oct 2026 | **Lead capture and lead scoring built**: private lead addresses (Google Ads form, Facebook via Zapier/Make, IndiaMART, JustDial, website forms) and a rules-based Hot / Warm / Cold score with reasons; no paid service, no AI. Direct Facebook Lead Ads and portal email parsing left for later | Closes the two gaps found against other CRMs ([30](30-lead-capture-and-scoring.md)) |
| 29 Oct 2026 | **Vibe Blueprint PDF used as the method for VBW CRM** (acceptance criteria, validation checklist, definition of done, 8-doc structure), not as a second product. Result: 81 integration tests and lint added to the repo and CI ([27](27-blueprint-validation-report.md), [28](28-testing-and-quality.md)) | Owner's decision; keeps one product and one stack |
| 28 Oct 2026 | **Clinic renamed Hospital; Retail Shop and Real Estate added**, each with its own fields, stages, messages, dashboard and screens (purchases; properties with client matching); **Offers** for every industry, sent one WhatsApp tap per person and only to customers who agreed. Replaces "clinics (small, not hospitals)" from 24 Oct | Owner's request: different CRM per industry; retail needs its own customer data for offers ([26](26-industries-retail-real-estate.md)) |
| 31 Oct 2026 | **Plan lengths, calendar dates and a "Due soon" screen** for renewals and follow-ups in every industry (gym 1–18 months, hospital/dental days, real estate 90/60/30 before a rent agreement ends, retail repeat and service dates), several reminder points, upgrade at renewal, a push to the owner. Reminders reach the owner, not customers. See [36](36-design-plans-and-due-soon-reminders.md), [38](38-due-soon-reminders-built.md) | Owner's decision |
| 30 Oct 2026 | **Fixed reports** (by source, stage, person; Excel) and **built-in automations** (hot lead and quiet lead tasks, who gets new-lead tasks; all off by default). No report builder, no rule builder, no messages to customers. See [33](33-reports-and-automations.md) | Owner's decision |
| 30 Oct 2026 | **Price ₹599 per account per month + GST** (was ₹399); **storage 20 GB per account** (was 4 GB), shared by both people. Every person (Owner and Member) can attach reports, scans and other files to a customer or lead, on web and Android, also while adding them. See [32](32-files-and-reports-price-599.md) | Owner's decision |
| 28 Oct 2026 | **Developer API** for each business (owner-made API keys, read-only or read-and-change, plus a signed webhook) so clients can connect AI calling and AI booking; no extra cost to us | Owner's request: clients want AI features through our app ([25](25-developer-api.md)) |
| 27 Oct 2026 | **Online booking page per business** with "Online booking by VBW CRM" name tag; leads tagged by source (ads, WhatsApp, QR, website, Google); pop-up reminder on opening the app | Owner's request: more leads, nothing missed, our name on every client's page ([24](24-online-booking-and-leads.md)) |
| 27 Oct 2026 | **Storage 4 GB per account**, shared by both people (was 2 GB per person); **trial 100 days for every new account** (was 14, with 100 only for pilots). The trial ends at the exact moment (sign-up or activation time + 100 days), then the account is view-only until paid | Owner's decision |
| 27 Oct 2026 | **Price ₹399 per account per month + GST** (was ₹499); one constant `PRICE_PAISE` in `packages/core` drives every price shown | Owner's decision |
| 27 Oct 2026 | Pilot: **30 accounts, 100 days free** (`TRIAL_DAYS=100`); replaces the 60-day decision below | Owner's decision; more time for pilots to see value |
| 27 Oct 2026 | Go straight to cloud + GitHub; skip testing real keys on the local computer | Owner's decision; real keys are tested on the server instead ([22](22-cloud-plan.md)) |
| 27 Oct 2026 | Hosting re-planned: Hetzner raised 2026 prices (4 vCPU/8 GB Singapore now €49.49/month) | See [22 — Cloud plan](22-cloud-plan.md) |
| 24 Oct 2026 | Keep code (`vbw-crm`) and documents (`vbw-crm-docs`) in separate folders | Easy to tell the software apart from the plans |
| 24 Oct 2026 | Phase 1 payments by UPI/bank with manual activation; payment buttons hidden until Razorpay | Launch faster; Razorpay KYC takes time |
| 24 Oct 2026 | Pilot trial 60 days (`TRIAL_DAYS` setting) | Pilots need time to see value |
| 24 Oct 2026 | Launch the web app first; Android app later | Smaller first step; web works on phone browsers |
| 24 Oct 2026 | Add Razorpay in phase 2 | Built already; switched on with keys |
| 24 Oct 2026 | First industries: clinics (small, not hospitals), gyms, education centres | Same core needs: appointments + renewals |
| 24 Oct 2026 | Company VBW, product name VBW CRM; domain already owned | — |
| 24 Oct 2026 | Hosting: Hetzner Singapore main + Contabo Mumbai standby + Cloudflare R2, about ₹2,300–2,900/month | Budget limit ₹2–3K/month without losing reliability |
| 24 Oct 2026 | No hiring: partners build and sell; budget covers technology only | Company plan |
| 24 Oct 2026 | Login with Firebase Authentication (email/password, branded reset, Google) | Proven service; no password code written by us |
| 24 Oct 2026 | Account = 2 people (Owner + Member), ₹499/month + GST, 2 GB per person | Simple and affordable |
| 24 Oct 2026 | 5-minute setup with industry templates | Customers are not technical |
| 23 Oct 2026 | Android only (no iOS) for now | Most target customers use Android |
| 23 Oct 2026 | Focus on service businesses (clinics, gyms, education, retail, real estate) | Clear focus; avoid enterprise complexity |
| 23 Oct 2026 | Modular monolith (one API), not microservices | Small team, lower cost |

## Open questions

| Question | Who decides |
|---|---|
| Exact Hetzner Singapore and Contabo Mumbai prices (sources disagree) | Check on provider sites before buying |
| How the DPDP Act applies to data stored in Singapore | Lawyer |
| GST on foreign services (Hetzner, Contabo) | Accountant |
| Support WhatsApp number to show customers | Owner |
