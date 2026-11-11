# VBW CRM — Documentation

**VBW CRM is a simple, real-time CRM for Indian businesses: customers, follow-ups, appointments and WhatsApp in one place, on web and Android.**

This folder holds **all planning, business and architecture documents**.
The **code** lives separately in [`../vbw-crm`](../vbw-crm).

```
car web/
├── vbw-crm/        ← CODE (the software: API, web app, Android app, server setup)
└── vbw-crm-docs/   ← DOCUMENTS (this folder: what we build, why, and how it fits together)
```

## Read in this order

| # | Document | What it answers |
|---|---|---|
| 1 | [Problem and solution](01-problem-and-solution.md) | Why VBW CRM exists; what is wrong with today's CRMs |
| 2 | [Customers and pricing](02-customers-and-pricing.md) | Who we sell to, ₹599 plan, storage, trial |
| 3 | [Features](03-features.md) | Everything the product does, per industry |
| 4 | [Architecture](04-architecture.md) | How the parts fit together (diagram) |
| 5 | [Technology stack](05-technology-stack.md) | Every tool we use and why |
| 6 | [Hosting and monthly cost](06-hosting-and-cost.md) | Servers, free tools, ₹2,300–2,900 a month |
| 7 | [Security and reliability](07-security-and-reliability.md) | Data protection, backups, speed targets |
| 8 | [Roadmap](08-roadmap.md) | Phase 1 (web), phase 2 (payments), phase 3 (Android) |
| 9 | [Go-to-market](09-go-to-market.md) | How we find the first customers |
| 10 | [Operations](10-operations.md) | Day-to-day: activating customers, backups, deploys |
| 11 | [Decisions log](11-decisions-log.md) | Every decision made so far, with dates |
| 12 | [Launch checklist](12-launch-checklist.md) | Every step from creating accounts to the first pilot customer |
| 13 | [Call script](13-call-script.md) | What to say on sales calls, in English and Tamil, with replies to common objections |
| 14 | [Demo video script](14-demo-video-script.md) | The 2-minute demo video, scene by scene, English + Tamil |
| 15 | [Brochure copy](15-brochure-copy.md) | Text for the one-page brochure, with hospital / gym / coaching variants |
| 16 | [Target list template](16-target-list-template.csv) | Spreadsheet for the 100 businesses to call (open in Excel or Google Sheets) |
| 17 | [Terms, Privacy, DPA: draft](17-terms-privacy-draft.md) | **Draft for the lawyer:** Terms of Service, Privacy Policy, Data Processing Agreement, sub-processors, DPDP facts, 12 questions |
| 18 | [Claude Design prompts](18-claude-design-prompts.md) | Steps and 4 prompts to design every screen of the app in Claude Design, with a feature checklist |
| 19 | [Login pages + animation prompt](19-login-pages-design-prompt.md) | Combined Prompt 2: landing, login, sign-up, password, invite, setup + the VBW Pulse animation and split-screen layout |
| 20 | [Handover for a new conversation](20-handover-new-conversation.md) | Everything a new Claude conversation needs to continue the work |
| 21 | [Step-by-step guide](21-step-by-step-guide.md) | **Start here for what to do next:** run locally, push to GitHub, then every launch step in order |
| 22 | [Cloud plan](22-cloud-plan.md) | Hosting for 30 pilot accounts with 100 days free: DigitalOcean Bangalore + Contabo Mumbai, every service to buy, ₹ per month, the cloud-first process |
| 23 | [Data safety and security](23-data-safety.md) | CIA + AAA review: clients cannot see each other (tested), restore deleted data, one-client restore, backups; 10 gaps found, 8 fixed |
| 24 | [Online booking and leads](24-online-booking-and-leads.md) | Booking page per business, top 5 lead channels, lead sources, pop-up reminder, and the list of key features still missing |
| 25 | [Developer API](25-developer-api.md) | API keys and webhooks so a client's AI voice agent or chatbot can recognise callers, book appointments and log calls; security and test results |
| 26 | [Hospital, retail, real estate, offers](26-industries-retail-real-estate.md) | Clinic renamed Hospital; Retail and Real Estate added with their own fields, purchases, properties and matching; WhatsApp offers only to customers who agreed; research sources and tests |
| 27 | [Blueprint validation report](27-blueprint-validation-report.md) | VBW CRM checked against the Vibe Blueprint method: checklist, MVP acceptance, docs coverage, gaps and next steps |
| 28 | [Testing and quality](28-testing-and-quality.md) | The five checks and how to run them, the 108 integration tests (what they cover, how to add one), lint, and the CI pipeline |
| 29 | [Storage plan](29-storage-plan.md) | Everything stored for the 30-account pilot, measured and estimated: database, files, backups, logs, server disks, monthly growth and cost, three risks found, monthly checklist |
| 31 | [Launch gaps: browser tests, accessibility, security policy, monitoring](31-launch-gaps-browser-tests-monitoring.md) | File-quota bug fixed, vulnerabilities 43 → 15, 28 real-browser tests, accessibility checks, Content Security Policy, health/metrics/alerts, and the drills that still need servers |
| 32 | [Files and reports, ₹599, 20 GB](32-files-and-reports-price-599.md) | Reports and scans per customer on web and phone (also while adding a lead), new price ₹599, 20 GB per account, what it costs |
| 33 | [Fixed reports and built-in automations](33-reports-and-automations.md) | Reports screen (source, stage, per person, Excel), hot and quiet lead tasks, who gets new-lead tasks, a browser bug fixed |
| 34 | [Plan for 100 accounts](34-plan-for-100-accounts.md) | Servers, database, storage and monthly cost for 100 accounts and 300 people online, with the triggers for scaling up |
| 35 | [Complete system overview](35-complete-system-overview.md) | Every tool, front end, back end, all 127 API routes, jobs, tables, screens, tests, and what is done and not done |
| 36 | [Design: plan lengths, calendar dates, Due soon](36-design-plans-and-due-soon-reminders.md) | Design for the reminders feature (approved and built) |
| 37 | [Implementation plan: Due soon reminders](37-plan-due-soon-reminders.md) | Six build tasks with interfaces and tests for the approved design |
| 38 | [Due soon reminders: what was built](38-due-soon-reminders-built.md) | How to use plan lengths, calendar dates and the Due soon screen per industry; tests; honest limits |
| 40 | [Safe deploys with no downtime](40-safe-deploys.md) | Images built and tested in GitHub Actions, 2 copies of API and web, swap without an outage, one-time setup steps |
| 30 | [Lead capture and scoring](30-lead-capture-and-scoring.md) | Lead addresses (Google Ads, Facebook via Zapier/Make, IndiaMART, JustDial, website forms), Hot / Warm / Cold score with reasons, what was tested and what was not |

## Status at a glance (27 Oct 2026)

| Area | Status |
|---|---|
| Planning and documents | ✅ Done |
| Code: API, background jobs, web app, Android app, server setup | ✅ Built and tested on a local computer |
| Phase 1 billing (manual UPI/bank activation) | ✅ Built and tested |
| Backend security hardening | ✅ Done (24 Oct 2026) |
| Phone-width check of all web screens (3 industries) | ✅ Done, 4 issues fixed (24 Oct 2026); real-phone test still to do |
| New design: every screen click-tested on phone, tablet, desktop, light + dark | ✅ Done, 6 issues fixed (27 Oct 2026) |
| File upload/download/delete with local storage stand-in | ✅ Tested (27 Oct 2026) |
| Code saved in git | ✅ 12 commits; ⏳ not on GitHub yet ([21](21-step-by-step-guide.md), Part 2) |
| Terms of Service + Privacy Policy pages | 📝 Draft with the lawyer ([17](17-terms-privacy-draft.md)); pages, sign-up checkbox and grievance contact built, showing the draft |
| Data retention, account deletion, 1-year access log, breach procedure | ✅ Built (24 Oct 2026); unpaid-account erasure switched off until the lawyer confirms |
| Real service accounts (Firebase, Brevo, Cloudflare R2) | ⏳ Not created yet |
| Servers (Hetzner + Contabo) | ⏳ Not bought yet |
| Pilot customers | ⏳ Not started |

The full plan is also in the Claude Doc "VBW CRM – End-to-End Plan".
