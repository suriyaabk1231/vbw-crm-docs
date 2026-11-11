# 10. Operations — day-to-day tasks

Technical setup steps (install, run, deploy) are in the code folder: [`../vbw-crm/README.md`](../vbw-crm/README.md).

## Activating a customer who paid by UPI / bank transfer (phase 1)

1. Customer pays ₹599 + GST and sends the payment details on WhatsApp.
2. Check the payment arrived in the bank/UPI app.
3. Run on the server (replace the email and reference):

```bash
docker compose -f infra/docker-compose.yml run --rm migrate node dist/db/activate.js --email owner@hospital.in --months 1 --note "UPI ref 412345678901"
```

4. Reply to the customer: "Your VBW CRM is active until <date shown by the command>."

| Need | Command option |
|---|---|
| See an account's status | `--email owner@hospital.in` (nothing else) |
| Activate / renew 1 month | `--months 1` |
| Renew several months | `--months 3` |
| Give a pilot more trial time | `--trial-days 30` |

- Renewing early adds on to the current end date.
- When the paid period ends, the account becomes view-only (nothing is deleted). Owners get reminder emails 3 days and 1 day before.
- Every activation is in the account's activity log.

## Deploying a new version

```bash
/opt/vbw-crm/infra/scripts/deploy.sh
```

Pulls the latest code, rebuilds, runs database updates, restarts. Then check `https://<api domain>/api/v1/health`.

## Backups

| Task | Command (on the main server) |
|---|---|
| First time setup | `infra/scripts/backup.sh init` |
| Automatic (cron) | `backup.sh full` on Sundays, `backup.sh diff` other days, 2:00 AM |
| See backups | `infra/scripts/backup.sh info` |
| Monthly restore test | `infra/scripts/restore-test.sh` |

## If the main server is down

1. Check UptimeRobot / Hetzner status. If it won't be back soon:
2. On the standby: `infra/standby/promote.sh` (type `PROMOTE`).
3. Start the app containers on the standby with the same `infra/.env`.
4. Point the Cloudflare DNS records (app + api) to the standby's IP.
5. Later, rebuild a new standby — never restart the old main database as primary.

## Support

- Customers message the support WhatsApp number (`SUPPORT_WHATSAPP`).
- Aim to reply within 2 hours on working days.
- Common fixes: device logged out (they logged in elsewhere), plan expired (activate), storage full (delete old files).

## Data retention (automatic, nightly jobs in the worker)

| What | When | Job |
|---|---|---|
| Contacts a business deleted | Erased for good 30 days after deletion, with their files | `purge-deleted-contacts` (3:15 AM) |
| Owner pressed **Delete account** (Settings → Data) | Erased 7 days later unless they cancel; the owner gets an email at both points | `data-retention` (3:30 AM) |
| Account unpaid after its plan ended | Warning emails 30 days and 48 hours before, erased 90 days after the plan ended. **Only when `LAPSED_ACCOUNT_ERASURE=true`** (keep `false` until the lawyer confirms the period). | `data-retention` |
| API access log | Daily file, compressed and copied to R2 `access-logs/` each night; local copy kept 30 days, R2 copy 1 year | `archive-access-logs` (12:30 AM) |

- Every erased account leaves one row in `erased_businesses`: id, date, reason and counts. It holds no names or contact details.
- To stop an erasure (for example the customer paid just now): activate them with `npm run activate`. A renewed account gets a new erasure date and the old warnings no longer count. For an owner request: Settings → Data → **Cancel deletion**, or set `deletion_requested_at = NULL`.
- A customer asks for deletion by WhatsApp or email instead of in the app: ask the **owner** to press Delete account, or after confirming it is really the owner, run as the owner role:
  `UPDATE businesses SET deletion_requested_at = now() WHERE id = '<business id>';`

## Personal data breach (DPDP Rules, Rule 7)

A **breach** is any unauthorised access to, or loss, change or disclosure of personal data. Examples: a stolen server key or laptop with
the `.env`, a database dump left public, a bug that showed one business another's contacts, a lost backup password, or a hacked Brevo or Firebase account.

**Who decides:** [Name 1] (first contact), [Name 2] (backup). Either of them can start this procedure; do not wait for certainty.

### The clock

| When | What | Who |
|---|---|---|
| **Hour 0**: someone notices | Write down the time you became aware. Open a private note "Breach [date]" and log every step with times. | Whoever noticed |
| **Within 1 hour** | **Contain:** rotate the leaked key or password (DB, Redis, R2, Brevo, Firebase, SSH); block the attacker's IP in Cloudflare; revoke sessions if needed (`UPDATE devices SET revoked_at = now()`); take the server offline if data is still leaking. Keep evidence: do not delete logs. | Decider |
| **Within a few hours** | **Find out:** which businesses and which data (the access log in `ACCESS_LOG_DIR` / R2 `access-logs/`, `audit_log`, Caddy logs). How many people? Which fields? | Decider + partner |
| **Without delay** | **Tell the Data Protection Board:** what happened, extent, time, place, likely impact (Rule 7(2)(a)). | Decider |
| **Without delay** | **Tell each affected business owner** (they are the Data Fiduciary for their customers; our DPA promises within [24] hours). For **our own users' data**, tell each affected user directly (Rule 7(1)). | Decider |
| **Within 72 hours** of becoming aware | **Detailed report to the Board** (Rule 7(2)(b)): updated details; facts, circumstances and reasons; mitigation done or planned; findings about who caused it; steps to prevent it happening again; a report of the messages sent to affected people. If you need longer, ask the Board in writing before 72 hours. | Decider |
| After | Fix the cause, add a test or check, and add a line to [11 — Decisions log](11-decisions-log.md). | Both |

> Rule 7 comes fully into force with the 18-month phase (about May 2027). Follow it from launch anyway.
> The Board's reporting channel will be on its portal: *[lawyer to confirm the address and format]*.

### Message to an affected person or business owner (Rule 7(1) asks for each point)

> **Subject: Important: a security problem affected your VBW CRM data**
>
> On [date and time] we found that [plain description: what happened, which data, how many records].
> **What this may mean for you:** [likely consequences, for example "someone may have seen your customers' names and phone numbers"].
> **What we have done:** [steps taken: key changed, access blocked, the bug fixed].
> **What you can do:** [for example "watch for unusual WhatsApp messages to your customers; change your password"].
> **Questions:** [name], [phone], [email]: we will answer the same day.
> — VBW

### Do not
- Do not delete logs or wipe the server before the facts are written down.
- Do not tell customers "nothing was affected" before you have checked.
- Do not post about it publicly before affected people and the Board have been told.
