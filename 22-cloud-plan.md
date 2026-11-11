# 22 · Cloud plan: 30 pilot accounts, 100 days free (27 Oct 2026)

**Decision asked:** go straight to the cloud and GitHub (no more testing of real keys on the local computer), on the smallest
budget that keeps every feature, the user experience, security and login exactly as built.

**In one line:** one main server in **DigitalOcean Bangalore**, a small standby in **Contabo Mumbai**, and free tiers for
everything else (Cloudflare, R2, Firebase, Brevo, GitHub). About **₹3,550–3,750 a month before GST**, or **₹2,850**
if the standby is added only after the pilot. Nothing in the app changes.

Prices below were checked on 27 Oct 2026. Conversions use **₹96.5 per US$** and **₹110 per €** (*estimates*: the sources give no
dated October rate). Foreign cloud providers usually add **18% GST** for Indian accounts: check on your first invoice.

---

## 1. What changed since doc 06

| | Doc 06 said | Today (27 Oct 2026) |
|---|---|---|
| Hetzner Singapore, 4 vCPU / 8 GB | ₹1,400–1,800 / month | **€49.49 ≈ ₹5,440** (CPX32). Hetzner raised prices in 2026, and its cheaper "Cost-Optimized" plans show *currently unavailable*. |
| Hetzner Singapore, 2 vCPU / 4 GB | — | **€26.99 ≈ ₹2,970** (CPX22), only 1 TB traffic included |

So the Hetzner + Contabo plan in doc 06 now costs about ₹6,000+ a month. This document replaces it for the pilot.

---

## 2. How much server do 30 accounts need?

- 30 accounts × 2 people = **60 people**. Usually 20–40% are online together (*rule of thumb from doc 06, not measured*):
  about **12–24 people at once**.
- The app was designed and load-tested in plan for 300 people at once on 4 vCPU / 8 GB.
- *Estimate:* 2 vCPU / 4 GB comfortably runs PostgreSQL, Redis (capped at 512 MB), API, worker, web and Caddy for 60 people.
  It is **checked** in step 10 below with the k6 load test, and the server can be resized in minutes if memory stays above 80%.
- Building the web app on a 4 GB server needs swap memory: `harden-server.sh` now adds 4 GB of swap automatically (27 Oct 2026).

---

## 3. Options compared

| Option | Where | Size | ₹ / month (before GST) | Verdict |
|---|---|---|---|---|
| **A. DigitalOcean (main)** | Bangalore, India | 2 vCPU / 4 GB / 80 GB, 4 TB traffic | $24 + weekly backups 20% ($4.80) = **≈ ₹2,780** | **Chosen.** Official price page, data in India, low delay for Indian users, resize in minutes. |
| A+. DigitalOcean (bigger) | Bangalore | 4 vCPU / 8 GB / 160 GB | $48 + 20% = ≈ ₹5,560 | Only if the load test fails on 4 GB. |
| **B. Contabo (standby)** | Mumbai, India | 4 vCPU / 8 GB / 100 GB | €4.40–5.50 ≈ ₹485–605, **plus an India location fee** (amount not published; confirm in the order form) | **Chosen as standby** (live copy of the database). |
| C. Hetzner | Singapore | 2 vCPU / 4 GB | €26.99 ≈ ₹2,970 | Dearer than A, outside India, only 1 TB traffic. |
| D. Contabo only (main + standby) | Mumbai | 4 vCPU / 8 GB each | ≈ ₹1,000–1,200 + location fees | Cheapest. Doc 06 notes reviews of slowdowns from shared hardware: risky as the main server. |
| E. Hostinger KVM 2 | India | 2 vCPU / 8 GB | ₹799 promo, ₹1,199 renewal (*third-party figures, not confirmed on Hostinger's site*) | Possible later; promo needs a 2-year commitment. |
| F. Managed database (DigitalOcean) | — | 1 GB | about $15 (*third-party figure*) | Not needed: our own PostgreSQL already has row-level security, continuous encrypted backups and a standby. |
| G. Supabase / Neon / Oracle free tiers | — | — | ₹0 | Not for paying customers (doc 06). Also cannot run our Redis + background worker. |

### Two budget levels

| | Recommended | Lowest (pilot only) |
|---|---|---|
| Main server + weekly snapshot (DigitalOcean) | ₹2,780 | ₹2,780 |
| Standby (Contabo Mumbai) | ₹600–800 (*estimate incl. location fee*) | — (add when the first customer pays) |
| Files + backups (Cloudflare R2) | ₹0 (10 GB free) | ₹0 |
| Domain (.in) | ≈ ₹75 (₹549 first year, ≈ ₹899 renewal) | ≈ ₹75 |
| Everything else (section 4) | ₹0 | ₹0 |
| **Total per month, before GST** | **≈ ₹3,550–3,750** | **≈ ₹2,850** |
| With 18% GST (*if charged*) | ≈ ₹4,200–4,450 | ≈ ₹3,370 |
| **For the 100-day pilot (≈ 3.3 months)** | **≈ ₹14,000–14,800** | **≈ ₹11,200** |

**What the lowest level gives up:** only *how fast* the app comes back if the main server dies. Data is still safe:
every database change is copied, encrypted, to R2 continuously (`backup.sh`), plus DigitalOcean's weekly snapshot.
Recovery without a standby means restoring onto a new server: *estimate* 1–2 hours instead of 5–15 minutes. Features,
security and login are the same in both.

---

## 4. Third-party services: what to buy and what is free

| Service | Used for | Plan | Cost | Limit that matters for 60 people |
|---|---|---|---|---|
| **DigitalOcean** | Main server (Bangalore) | Basic Droplet 2 vCPU / 4 GB + weekly backups | **$28.80 / month** | — |
| **Contabo** | Standby server (Mumbai) | Cloud VPS 4 | **≈ €4.40–5.50 / month + location fee** | — |
| **Domain registrar** | `<name>.in` | 1 year | **≈ ₹549** first year | — |
| Cloudflare | DNS, HTTPS certificate, firewall, DDoS protection | Free | ₹0 | — |
| Cloudflare R2 | Customer files + database backups | Free tier | ₹0 | 10 GB-month, 1 M writes, 10 M reads free; then $0.015/GB. No download fees. Worst case if all 30 pilots filled their 20 GB: 600 GB of files + 600 GB backup copy ≈ $18 ≈ ₹1,600 / month (estimate, doc 32) |
| Firebase Authentication | Login: email + password, Google | Spark (free) | ₹0 | 50,000 monthly users free. **SMS/phone login is not free** and is not used. |
| Firebase Cloud Messaging | Push notifications (Android app, later) | Free | ₹0 | — |
| Brevo | Password-reset and invite emails | Free | ₹0 | **300 emails a day** (*third-party sources; confirm on brevo.com*). 60 people need a few a day. Free plan shows Brevo branding. |
| GitHub | Private code storage + automatic tests | Free | ₹0 | 2,000 Actions minutes a month; each test run is about 5 minutes |
| UptimeRobot, Sentry | Downtime alerts, error reports | Free | ₹0 | (*from doc 06; not re-checked today*) |
| Later: Razorpay | Online payments (phase 2) | Pay per use | ≈ 2% per payment | — |
| Later: Google Play | Android app (phase 3) | One time | ₹2,100 | — |

Nothing else needs to be bought for the pilot.

---

## 5. What stays exactly as built (no sacrifice)

| Area | Kept |
|---|---|
| Features | All of doc 03 and doc 20 section 4, all 3 industries |
| Login | Firebase email + password and Google; 1 phone + 1 browser per person; owner/member roles |
| Data isolation | PostgreSQL row-level security per business (tested by GitHub on every push) |
| Network | Cloudflare in front (HTTPS, firewall, DDoS); server accepts web traffic **only** from Cloudflare; SSH by key only; fail2ban; automatic security updates |
| Backups | Continuous, encrypted to R2, plus weekly snapshot; monthly restore test |
| Privacy (DPDP Act 2023) | Data stored in India (Bangalore + Mumbai); deletion, access log, grievance officer, legal pages |
| Speed | Indian data centre; checked by the load test before pilots |

---

## 6. 100-day free pilot for 30 accounts: how it works in the app

1. On the server, `TRIAL_DAYS=100` in `infra/.env` (already set in `infra/.env.example`). The home page, sign-up page and Terms
   all say 100 days automatically.
2. Every account that signs up while this is set gets 100 days free. The app shows "trial ending" notices in the last 7 days.
3. 100 days is now the trial for every new account, not only pilots (decided 27 Oct 2026). A single account can
   still be given 100 days later with `activate --email <owner> --trial-days 100`.
4. On day 100 the account becomes view-only (nothing is deleted; export still works). When they pay ₹599 + GST by UPI:
   `activate --email <owner> --months 1 --note "UPI ref"`.
5. Unpaid-account deletion stays **off** (`LAPSED_ACCOUNT_ERASURE=false`) until the lawyer confirms.

Changed on 27 Oct 2026 from 60 to 100 days in: docs 02, 08, 09, 12, 13 (English + Tamil), 14, 15, 17, 21, the Terms draft
(`apps/web/content/legal/terms.md`), `infra/.env.example` and the `activate` help text. Logged in doc 11.

---

## 7. End-to-end process (cloud first)

This replaces Part 4 of [21](21-step-by-step-guide.md) (testing real keys on the computer): the same checks are done on the
live server in step 8. All other details are in doc 21 and doc 12.

| # | Step | Who | Done when |
|---|---|---|---|
| 1 | Push code and docs to 2 **private** GitHub repositories (doc 21, Part 2) | you | GitHub **Actions** shows a green tick |
| 2 | Buy the domain; add it to Cloudflare; change nameservers; SSL mode Full (strict); create the origin certificate (doc 21, 3.1) | you | Cloudflare says *Active* |
| 3 | R2: buckets `vbw-crm-files` + `vbw-crm-backups`, API token, CORS for `https://app.<domain>` (doc 21, 3.2) | you | keys saved in a password manager |
| 4 | Firebase project: Email/Password + Google on; add `app.<domain>`; web app values; service-account key (doc 21, 3.3) | you | 4 web values + JSON key saved |
| 5 | Brevo: domain verified (SPF, DKIM, DMARC), sender `no-reply@<domain>`, API key (doc 21, 3.4) | you | Brevo shows the domain verified |
| 6 | **Buy servers:** DigitalOcean → Create Droplet → region **Bangalore** → **Ubuntu 24.04 LTS** → Basic, Regular, **2 vCPU / 4 GB** → add your SSH key → tick **Backups (weekly)**. Contabo → Cloud VPS 4 → region **India** → Ubuntu 24.04 → SSH key (or skip Contabo at the lowest budget level) | you | you can `ssh root@<ip>` with your key |
| 7 | On each server: clone the repo to `/opt/vbw-crm`; run `infra/scripts/harden-server.sh` (now also adds swap); WireGuard; install Docker; run `harden-server.sh` once more so the Docker firewall rule is added (doc 21, Part 5 steps 2–6) | Claude can guide live | `ssh` works; `http://<ip>` does **not** load |
| 8 | Fill `infra/.env` (keys, passwords, `TRIAL_DAYS=100`, support WhatsApp, grievance officer), save the certificate, Cloudflare DNS `app` + `api` proxied; `docker compose up -d --build`; then **the real-key checks on the live site**: sign-up, forgot password email, Google login, file upload/download/delete, invite email, 1-browser rule, WhatsApp button (doc 21, Parts 5–6) | you + Claude | every check passes on `https://app.<domain>` |
| 9 | Backups: `backup.sh init`, `backup.sh full`, cron lines; **restore test**; standby replica; switch-over practice (doc 21, Part 7) | you + Claude | restore row counts match |
| 10 | **Load test** for 60 people (k6, on the standby before it becomes a replica): 95% of requests under 300 ms, under 1% failing (doc 21, Part 8) | Claude can run | pass; if not, resize the Droplet to 4 vCPU / 8 GB |
| 11 | **Automatic deploy from GitHub** (to build): after the tests pass on `main`, GitHub logs in to the server with a deploy-only SSH key and runs `infra/scripts/deploy.sh`. Until then, deploy by running `deploy.sh` on the server | Claude | a push to `main` updates the live app |
| 12 | Before pilots: lawyer-approved legal texts; breach-procedure deciders; activation test; real Android + iPhone check; delete test accounts; uptime monitor (doc 21, Part 9) | you | all ticked |
| 13 | Onboard the 30 pilots (10 per industry): set up each one, import their Excel (doc 21, Part 10) | you | 30 accounts active |
| 14 | During the 100 days: weekly feedback calls; fix the top 3 complaints every 2 weeks; monthly restore test | you + Claude | — |
| 15 | Day 90–100: convert to paid (₹599 + GST by UPI, `activate --months 1`); add the standby now if you chose the lowest level | you | first payments in |

---

## 8. Still to confirm before buying (you)

- [ ] DigitalOcean: the Bangalore price at checkout matches $24 (the price page does not say whether prices differ by region).
- [ ] Contabo: the India location fee in the order form.
- [ ] Whether GST is added on each invoice.
- [ ] Brevo free plan limit on brevo.com.

## Sources (checked 27 Oct 2026)

- Hetzner Regular Performance prices incl. Singapore: <https://www.hetzner.com/cloud/regular-performance>; Cost-Optimized (unavailable): <https://www.hetzner.com/cloud/cost-optimized>
- DigitalOcean Droplet prices and backup add-on: <https://www.digitalocean.com/pricing/droplets>
- Contabo Cloud VPS: <https://contabo.com/en/vps/>; India location fee exists: <https://contabo.com/en-us/locations/india>
- Firebase pricing: <https://firebase.google.com/pricing>
- Cloudflare R2 pricing: <https://developers.cloudflare.com/r2/pricing/>
- Brevo free plan (third-party): <https://dreamlit.ai/blog/brevo-review>, <https://automationatlas.io/answers/brevo-pricing-explained-2026/>
- GitHub plans: <https://docs.github.com/en/get-started/learning-about-github/githubs-plans>
- .in domain prices (third-party): <https://domainoffer.net/tld/in>, <https://cybernews.com/best-domain-registrars/best-india-domain-registrars/>
- Hostinger VPS India (third-party): <https://zoutons.com/news/hostinger-vps-hosting-price-india-2026>
- DigitalOcean managed PostgreSQL (third-party): <https://infratally.com/articles/digitalocean-managed-postgres-deep-dive/>
- Exchange rates (*estimates*): <https://pluang.com/en/tools/currency-converter/usd-inr>, <https://30rates.com/eur-to-inr>, <https://www.poundsterlinglive.com/history/EUR-INR-2026>
