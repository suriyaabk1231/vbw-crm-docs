# 8. Roadmap

**In one line:** launch the **web app first** with manual (UPI/bank) payments, add **Razorpay** in phase 2,
publish the **Android app** in phase 3.

## Phases

| Phase | What | Status |
|---|---|---|
| **0. Plan** | Problem research, pricing, architecture, documents | ✅ Done |
| **0. Build** | API, worker, web app, Android app, server setup — built and tested locally | ✅ Done |
| **1. Web launch** | Real login (Firebase), email (Brevo), files (R2), servers, pilot customers; payments by UPI/bank with manual activation | ⏳ Next |
| **2. Online payments** | Razorpay subscription + customer payment links (already built; add keys and switch on) | ⏳ |
| **3. Android app** | Publish on Google Play (already built; needs device testing and Play review) | ⏳ |
| **4. Growth features** | WhatsApp Business API inbox, IndiaMART/JustDial lead capture, automation rules, Indian languages, more industries (Real Estate first) | ⏳ Not built |

## Phase 1 — step by step

| Week | Business side (owner / sales) | Developer side |
|---|---|---|
| 1 | Create Firebase, Brevo and Cloudflare accounts; build a list of 100 target businesses; first 30 calls | Push code to a private GitHub repository; add real keys; test real login, reset email, uploads |
| 2 | Book 10 pilots per industry (target 30) | Buy servers; harden them; set up WireGuard, Docker, Cloudflare, backups, standby |
| 3 | Make the 2-minute demo video (Tamil + English), brochure, landing page text | Load test (300 users), restore test, standby switch-over practice; fix issues |
| 4 | **Pilot launch:** set up each pilot personally, import their Excel | Monitor errors and speed daily; fix top complaints |
| 5–8 | Weekly calls with pilots; collect testimonials; convert to paid (₹599 by UPI) | Fix top 3 complaints every 2 weeks |

Pilot trial: 100 days (`TRIAL_DAYS=100`). Founder price for pilots: ₹599/month locked.

## Phase 2 — switching on payments

1. Complete Razorpay KYC; create the ₹599/month plan in the Razorpay dashboard.
2. Add `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET`, `RAZORPAY_PLAN_ID`; set `ONLINE_PAYMENTS=true`.
3. Add the webhook in Razorpay: `https://<api domain>/api/v1/webhooks/razorpay` (events `payment_link.paid`, `subscription.*`).
4. Rebuild and deploy. Subscribe and payment-link buttons appear automatically.

## Phase 3 — Android app

1. Test on 2–3 real Android phones (including a low-end one) on slow internet.
2. Add Firebase `google-services.json` for push notifications; set the real API address in `app.json`.
3. Google Play developer account (₹2,100), privacy policy URL, store listing, build with EAS, submit.
