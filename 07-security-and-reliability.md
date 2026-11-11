# 7. Security and reliability

**In one line:** we run our own servers, so these checks are our job. Tick them all before launch.

## Servers — `vbw-crm/infra/scripts/harden-server.sh`

- [ ] SSH keys only; password and root password login disabled
- [ ] Firewall: only ports 80/443, and only from Cloudflare's addresses
- [ ] Database and Redis never reachable from the internet
- [ ] WireGuard encrypted link between main and standby
- [ ] Automatic security updates and fail2ban
- [ ] Real server IP hidden behind Cloudflare

## Application (built in)

| Protection | How |
|---|---|
| No password code written by us | Firebase handles passwords; API checks the login token on every request |
| Businesses can't see each other | PostgreSQL Row-Level Security; tested automatically |
| Owner vs member | Owner-only actions checked on the server (billing, settings, export, team) |
| 2 people, 1 phone + 1 browser each | Enforced by the API; old device logged out |
| Abuse and password guessing | Request limits per person/IP (Redis); sign-up and reset limited per hour |
| Bad input | Every request validated (Zod) with readable messages |
| Secrets | Only in environment files, never in git (`.env` files are ignored) |
| Files | Private R2 bucket, time-limited links, only PDF/images/Excel, max 25 MB |
| Payments (phase 2) | Razorpay webhook signatures checked; each event processed once |
| Traceability | Audit log of changes, activations and sign-ups |
| Dev login | Only works in development; the API refuses to start with it on in production |

## Backups

- [ ] Continuous encrypted database backups to R2 (pgBackRest): weekly full, daily differential, kept about 30 days
- [ ] **Restore test before launch, then monthly** — `vbw-crm/infra/scripts/restore-test.sh`
- [ ] Switch-over to the standby practised once — `vbw-crm/infra/standby/promote.sh`

## Speed targets

| Measure | Target |
|---|---|
| API response (95% of requests) | under 300 ms |
| Search with 50,000 contacts | under 0.5 s |
| Web page load on 4G | under 2 s |
| People online at once | 300 (confirm with k6) |
| Uptime | 99.9% |

How we get there: database indexes on phone/name/stage, lists of 50 rows, dashboard cached in Redis (refreshed after
every change), photos compressed on the phone.

## Routine

| When | What |
|---|---|
| Weekly | Check alerts (Sentry, UptimeRobot) and disk space |
| Monthly | Server updates; restore test |
| Every release | Run tests; load test for big changes |

## Legal

- Terms of service and privacy policy written by a lawyer (needed for Google Play and the DPDP Act).
- Data is stored in Singapore (main) and Mumbai (standby) — confirm with a lawyer how the DPDP Act applies.
  The questions, the DPDP Rules timeline and the product gaps (G1–G8) are in [17 — Terms, Privacy, DPA draft](17-terms-privacy-draft.md).
- No medical records stored for hospitals.
