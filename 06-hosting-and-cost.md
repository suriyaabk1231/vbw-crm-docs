# 6. Hosting and monthly cost

**In one line:** about **₹2,300–2,900 a month** in technology costs, supporting about 300 people online at once.

> **Out of date (27 Oct 2026):** Hetzner raised its 2026 prices. For the pilot, use [22 — Cloud plan](22-cloud-plan.md).

All prices are from public pages and third-party sources (Oct 2026), assume about ₹100 per €1, and exclude GST.
**Check each provider's current price before buying.**

## Servers and storage

| Item | Provider | Size | ₹ / month |
|---|---|---|---|
| Main server (API, worker, Redis, PostgreSQL, web) | Hetzner Cloud, Singapore | 4 vCPU / 8 GB | 1,400–1,800 |
| Server snapshot backups | Hetzner add-on | — | 300 |
| Standby server (live database copy) | Contabo, Mumbai | 4 vCPU / 8 GB | 450–500 |
| Files + database backups | Cloudflare R2 | 10 GB free, then about ₹1.3/GB | 0–200 at start |
| Domain | Any registrar (₹1,200/year) | — | 100 |
| **Total** | | | **about 2,250–2,900** |

## Free tools

Firebase Authentication and Cloud Messaging, Cloudflare (CDN, SSL, firewall), Brevo email, GitHub + Actions,
Sentry, UptimeRobot, k6, pgBackRest, WireGuard, Expo builds — all ₹0 on their free tiers at our size.

## Costs that are not monthly

| Item | When |
|---|---|
| Google Play developer account ₹2,100 | One time, when the Android app is published (phase 3) |
| Razorpay about 2% per payment | Phase 2, only when customers pay online |
| Storage above 10 GB, about ₹1.3/GB | Only as customers upload more; worst case (250 accounts × 20 GB = 5,000 GB) about ₹6,500/month (estimate, see doc 32) |

## Capacity

- The plan targets **300 people online at once** (250 accounts × 2 people ≈ 500 people; usually 20–40% online together — a rule of thumb, not measured).
- Confirm with the k6 load test (`vbw-crm/infra/load-test/k6.js`) on a staging copy before launch.
- If the database becomes the bottleneck, upgrade the database resources (about +₹2,500/month).

## Options considered

| Option | ₹ / month | Verdict |
|---|---|---|
| **Hetzner + Contabo (chosen)** | 2,300–2,900 | Best balance of cost and reliability |
| Contabo only (main + standby) | 1,100–1,400 | Cheapest; reviews mention slowdowns from shared hardware |
| DigitalOcean Bangalore + managed database | about 9,000 | Easiest to run, over budget |
| Oracle Always Free / Supabase free | 0 | Not safe for paying customers |

## If the main server fails

Promote the Mumbai standby (`vbw-crm/infra/standby/promote.sh`, about 5–15 minutes) and point Cloudflare at it.
Data loss is at most a few seconds to minutes thanks to live replication and continuous backups.
