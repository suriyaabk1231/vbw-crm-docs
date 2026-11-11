# 34 — Plan for 100 accounts (up to 300 people online at once)

*Written 30 Oct 2026. Every number marked "estimate" is my calculation or a rule of thumb, **not measured**. Nothing here has been load-tested on a real server yet.*

## 1. The target

| | Number | How it was made |
|---|---|---|
| Accounts | **100** | Your decision (this document) |
| People | **200** | 100 accounts × 2 people |
| Online at once, normal | **40–80** | *Rule of thumb:* 20–40% of people online together (doc 06, not measured) |
| Online at once, design target | **300** | Doc 07 / doc 22: the app was planned for 300 people at once. This is 4–7 times the normal load, so it covers busy mornings, phone + browser logged in together, and a spike |
| Income at ₹599 | **₹59,900 a month** before GST | 100 × ₹599 |

Because 300 is more than the number of people (200), the target is generous on purpose. If the load test passes at 300, the real world will be comfortable.

## 2. What to use (same stack as the pilot, one size up)

| Part | Choice | Why |
|---|---|---|
| **Main server (VPS)** | DigitalOcean, Bangalore, **4 vCPU / 8 GB / 160 GB** + weekly backups | Doc 22 lists this as the step up from 2 vCPU / 4 GB, and the app was planned for 300 people online on this size |
| **Standby server** | Contabo, Mumbai, 4 vCPU / 8 GB / 100 GB | Live copy of the database; switch over if the main one fails |
| **Database** | **PostgreSQL**, on the main server | Tested, keeps each business separate at the database level (row-level security). **Not D1:** its 10 GB limit per database and the rewrite are not worth it at this size (see the earlier discussion) |
| **File storage** | **Cloudflare R2** | Scales without effort; no download fees |
| **Login** | **Firebase Authentication** | Free up to 50,000 monthly users (doc 22); 200 people is nothing |
| **DNS, HTTPS, firewall** | Cloudflare (free) | Already planned |

**Why not stay on 2 vCPU / 4 GB?** You can: start there for the first 30 accounts and **resize to 4 vCPU / 8 GB** (a few minutes on DigitalOcean) when the triggers in section 5 fire. If you would rather not think about it, start on 4 / 8 from the beginning; the extra cost is about ₹2,800 a month.

## 3. Monthly cost (estimate, before GST)

| Item | Cost |
|---|---|
| DigitalOcean 4 vCPU / 8 GB + weekly backups ($48 + 20%) | ≈ ₹5,560 (doc 22; recheck the price before buying) |
| Contabo standby | ≈ ₹485–605 + an India location fee (not published) |
| Domain `.in` | ≈ ₹549 a year (≈ ₹46 a month) |
| Cloudflare, Firebase | ₹0 |
| Cloudflare R2 | **₹60 – ₹430** (typical to heavy use; see section 4) |
| **Total** | **≈ ₹6,200 – ₹6,700 a month**, about **10–11%** of the ₹59,900 income |

Razorpay or other payment fees, email sending (Brevo) and error tracking (Sentry) are extra and not included.

## 4. Storage (scaled from doc 29's measurements for 30 accounts)

Doc 29 measured the database and estimated files for 30 accounts. Multiplying by 100/30 (about 3.3):

| | Typical | Heavy | Worst case (every account fills 20 GB) |
|---|---|---|---|
| R2 total (files + backup copy + database backups + logs) | ≈ 53 GB | ≈ 313 GB | ≈ 4,000 GB |
| R2 cost a month (first 10 GB free, $0.015 per GB after, ₹95 per $) | ≈ ₹60 | ≈ ₹430 | ≈ ₹5,700 |
| Server disk used (of 160 GB) | well under half | well under half | well under half |

"Typical" assumes about 0.24 GB of files per account; "heavy" assumes 1.5 GB per account (both from doc 29, assumptions). **The worst case is not realistic** for 100 accounts, but even then it is about ₹5,700 against ₹59,900 income. The only number that can surprise you is customers' files.

## 5. When to move up (triggers, not guesses)

Watch `/health/metrics` and the server graphs. Act when you see these for several days, not for one spike:

| You see | Do this |
|---|---|
| Server memory above 80% | Resize the main server (or move Redis/Postgres settings up) |
| CPU above 70% at busy hours | Resize to a bigger server |
| 95% of answers slower than 300 ms | Check slow database queries first (cheap), then resize |
| Database connections close to the limit (100) | Add a connection pooler (PgBouncer) before buying a bigger server |
| Disk above 70% | Grow the disk, or move older logs to R2 |
| About 300 accounts | Put the database on its own server; consider a read replica |

## 6. What must happen before this plan is real

1. **Buy the servers and deploy staging** (your side; doc 21 / 22).
2. **Run the load test** `infra/load-test/k6.js` at **300 people online**, target "95% of answers under 300 ms" (doc 07 / 31), while watching `/health/metrics`. Until then, section 2 is an estimate.
3. **Restore and failover drills** (doc 31).
4. **Add error tracking and an outside uptime check** so you see slowness before customers tell you.

## 7. Risks and honest limits

- **One main server is a single point of failure.** The standby reduces the damage but a switchover is a manual drill, not automatic. Fine for 100 accounts; revisit before 300.
- **The 300-online target is a design goal, not a result.** The k6 script exists but has never been run against a real server.
- **Prices change.** DigitalOcean, Contabo and R2 figures come from doc 22 and doc 29; confirm them on the day you buy. The Contabo location fee is unknown.
- **Payments are not in this table.** Razorpay (or another provider) fees depend on the plan and on payment method; see the payment discussion.

## 8. Decisions for you

1. Start on 4 vCPU / 8 GB straight away, or start on 2 vCPU / 4 GB and resize at the first trigger?
2. Is 100 accounts the right near-term target, or should the plan jump to 300 accounts (where the database needs its own server)?
