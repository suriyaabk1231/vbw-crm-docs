# 29 · Storage plan: everything that is stored, for the 30-account pilot (29 Oct 2026)

Question: how much storage does the whole system need, end to end, with the pilot (30 accounts, up to 60 people), including
sign-ups, logins, errors, emails, files, backups and logs, and how much does it grow each month.

**How the numbers were made.** *Measured* = I ran it on the real database schema (a year of busy use by 30 accounts loaded
into an empty copy, then measured) or on the real code. *Assumption* = my estimate, stated so you can change it. Prices are Cloudflare's
published R2 prices (<https://developers.cloudflare.com/r2/pricing/>, read 29 Oct 2026); the rupee figures use ₹95 to the dollar (estimate).

## 1 · The short answer

| | Light use | **Typical** | Heavy use | Worst case (every account fills its 4 GB) |
|---|---|---|---|---|
| Cloudflare R2 (files, copy of files, database backups, access logs) | 7 GB | **16 GB** | 94 GB | 244 GB |
| R2 cost a month (first 10 GB free) | ₹0 | **about ₹8** | about ₹120 | about ₹335 |
| Main server disk used (of the 80 GB included) | 15 GB | **17 GB** | 25 GB | 25 GB |
| Growth a month | 0.5 GB | **1.3 GB** | 7.5 GB | 10 GB |

"Typical" = every fifth customer has one file of about 0.8 MB, so about 0.24 GB of files per account. "Heavy" = 1.5 GB of files per account.
Nothing to buy beyond the plan in doc 22: the 80 GB server disk, the 100 GB standby disk and R2's pay-as-you-go cover all cases. The only
number that can surprise you is **customer files**; everything else is small.

## 2 · What is stored, and where

### 2.1 PostgreSQL: customers, history, appointments, tasks, users, sign-ups, devices, activity log (measured)

A busy account-year (1,500 customers, 6,000 history entries, 1,500 appointments, 900 follow-ups, 1,200 tasks, 4,500 activity-log
entries, 300 file records, 2 people, 20 browser logins) loaded 30 times:

| Table | Data + indexes, all 30 accounts |
|---|---|
| History (activities) | 50 MB |
| Customers | 38 MB |
| Activity log (audit_log, nobody can edit) | 26 MB |
| Appointments | 9 MB |
| Tasks | 7 MB |
| Follow-ups (renewals) | 5 MB |
| File records, devices, users, stages, templates, businesses | 5 MB |
| **Database total** | **148 MB** (of which 9 MB is the empty structure) |

So about **4.6 MB per account per year**, or about 12 MB a month for all 30. Three times busier (4,500 customers each): about 0.45 GB.
Five years at that rate: about 0.7 GB. The 4 GB per-account limit counts **files only**, never this data.

Sign-ups, logins, devices and invites are rows in these tables (a few hundred bytes each); they are included above.

### 2.2 Customer files (Cloudflare R2): the only big item

Limit: 4 GB per account, 25 MB per file. 30 accounts: **120 GB at most**. Typical use (assumption): 7 GB. Files deleted by a business stay 30 days
before they are erased for good, adding about 5 to 10 %. There is a **second copy** of every file in the backups bucket (doc 23, G2), so double it.
Uploads go straight to R2, never through the server's disk.

### 2.3 Database backups (R2, pgBackRest, encrypted, compressed)

Settings in `infra/postgres/pgbackrest.conf`: 4 weekly full backups, 14 daily differences, and every change as it happens (the log files
are cut every 5 minutes at most).

| Part | Measured / estimated | Size |
|---|---|---|
| One full backup | database 148 MB; compressed to 13 MB (measured, zstd) to 26 MB (measured, zlib dump). My test data repeats a lot, so I plan with a cautious 3 to 1 (about 50 MB) | 4 fulls ≈ 0.2 GB |
| Daily differences | assumption: 10 to 20 % of pages change a day | 14 × 10 MB ≈ 0.15 GB |
| Change log (WAL), kept about 28 days | **measured:** 480,000 rows written = 243 MB of log, compresses 4 to 1 (71 MB per 285 MB). A busy cluster of single clicks, measured worst case: 24.6 KB per scattered one-row update; best case about 0.5 KB per row. An almost empty 16 MB segment compresses to 8 KB, so the 5-minute switch costs about 1.3 MB a day at most | 0.02 to 1.0 GB, **central 0.35 GB** |
| **Total** | | **about 1 GB** (0.5 to 3 GB) |

### 2.4 Access log: every request, including failed logins that reach our API and error answers (R2, one year)

Measured: one line is **283 bytes**; a day compresses 9 to 1. At an assumed 1,000 lines per person per day (the app checks for news every 2
minutes, and every call has a browser pre-check line), 60 people write 17 MB a day, 1.9 MB compressed, **about 0.7 GB for a year** (0.3 to 1.4 GB
for 400 to 2,000 lines). Raw days are compressed and removed from the server each night; compressed copies stay on the server 30 days (about 60 MB).
If it is really 30 people rather than 30 accounts, halve this.

### 2.5 Things the user listed, one by one

| What | Where it is stored | Size |
|---|---|---|
| Sign-up | `users`, `businesses`, `devices` rows + a line in the activity log | under 1 KB each |
| Sign-in | Passwords and sign-in attempts are held by **Firebase (Google)**, not by us. Our server records each logged-in request in the access log. Each browser login is a `devices` row | Firebase: kilobytes for 60 people; ours: in 2.1 and 2.4 |
| **Login failures** | A wrong password never reaches our server (Firebase answers). A refused or logged-out session (401) does, and shows as a line in the access log | in 2.4 |
| **Error messages** | Written to each program's output, which Docker keeps as log files on the server. Nothing is stored in the database | estimate 0.4 to 1 MB a day; see the warning in section 4 |
| **Emails** (verification, password reset, invites, trial and deletion warnings) | Sent through Brevo's service; **we store no copy**, only the invite row. Brevo keeps its own sending log under its own rules | none on our side |
| Redis (rate-limit counters, 60-second dashboard cache, job queue) | In memory (capped at 512 MB) and a small file on disk | measured 3 MB now; allow 0.1 GB |

### 2.6 The servers' own disks

| Item | Main server (80 GB, DigitalOcean) |
|---|---|
| Ubuntu + packages | about 8 GB (estimate) |
| Docker images (database, Redis, API, web, Caddy) | about 3 GB; up to 8 GB with older copies and build cache (estimate; measured locally: database image 642 MB, Redis 170 MB) |
| Swap file | 4 GB (set by `harden-server.sh`) |
| Database files | 0.15 to 1 GB (measured, 2.1) |
| Database log files on disk (WAL) | 1 to 2 GB (settings: kept ≥ 1 GB for the standby) |
| Redis, access logs, container logs (now capped at 30 MB per program), deploy records | under 0.5 GB |
| **Total** | **about 17 GB typical, 25 GB busy** |

If the standby is down for a long time the main server keeps up to 10 GB of extra change log for it (the new cap), so the worst case is about 35 GB of 80 GB.

Standby server (Contabo, 100 GB): the same database and log files, about 13 GB. DigitalOcean's weekly snapshots are inside the plan's 20 % fee.
The pilot never needs a bigger disk.

## 3 · Monthly growth and what it costs

| What grows | Typical a month | Heavy a month |
|---|---|---|
| Customer files, and their copy | 0.6 GB × 2 = 1.2 GB | 3.75 GB × 2 = 7.5 GB |
| Access log (R2) | 0.06 GB | 0.1 GB |
| Database (server) | 0.012 GB | 0.04 GB |
| Backups | about steady (old ones expire) | about steady |
| **Total** | **≈ 1.3 GB** | **≈ 7.6 GB** |

R2 charges $0.015 a GB-month after the first 10 GB, nothing for downloads, and for requests only above 1 million "write" and 10 million "read"
operations a month. We use roughly 10,000 writes a month (estimate; about 8,600 of them log-file uploads), so requests cost ₹0.
After 12 months at typical use R2 holds about 16 GB (about ₹8 a month); at heavy use about 94 GB (about ₹120 a month).
Your planned 600 GB is for around 150 accounts, not for the pilot.

## 4 · Three things found while measuring: all fixed (29 Oct 2026)

| # | Problem | Fix | Checked |
|---|---|---|---|
| 1 | **Docker did not limit its log files.** A quiet day writes about 1 MB, but a repeating error could write 100 MB or more a day and fill the disk | `infra/docker-compose.yml` and `infra/standby/docker-compose.yml`: every program keeps at most **3 files of 10 MB** (30 MB), the oldest is dropped | The merged settings show the limit on all 7 programs of the main server and on the standby. A scaled-down run (limit 1 MB × 3 files) wrote about 9 MB and Docker kept 1.4 MB |
| 2 | **A stopped standby could fill the main disk**, because the main database keeps every change for it with no limit | `infra/postgres/postgresql.conf`: `max_slot_wal_keep_size = 10GB`. Past 10 GB the main server drops the standby's slot instead of filling its disk; the standby is then rebuilt with `standby/setup-replica.sh`. Backups to R2 are not affected | The real config file was read by PostgreSQL 16: `max_slot_wal_keep_size` = 10240 MB. A deliberately wrong value was refused, so the check really reads the file |
| 3 | **Docker build leftovers were not cleaned**, so the build cache grew with every deploy | `infra/scripts/deploy.sh`: after each deploy, `docker builder prune -f --filter "until=168h"` removes build cache older than a week (a failure here never stops a deploy) | Script passes shellcheck. **Not run on a real server yet** (no server exists), so confirm on the first deploys with `docker system df` |

Not changed, on purpose: the standby is still not watched automatically. Check it in the monthly list below, and watch disk space (alert at 70 %).
Optional, not applied: PostgreSQL's `wal_compression` setting shrinks the change log written after each checkpoint (see PostgreSQL's documentation);
worth testing if the backup bucket ever grows faster than planned.

## 5 · Monthly storage checklist (about 15 minutes)

1. Server disk: `df -h` (alert at 70 %); `docker system df`; then `docker builder prune` if build cache is large.
2. Container logs stay inside their limit (30 MB per program, section 4). If a program writes 30 MB in a day, read its log: something is repeating.
3. Standby is connected (`pg_stat_replication`); if not, fix it the same day. After 10 GB of changes without it, the main server drops its slot and the standby must be rebuilt.
4. Backups: `infra/scripts/backup.sh info` shows the sizes and the latest full backup; do the monthly restore test (doc 23).
5. R2 dashboard: total GB (compare with section 1), request counts, and the largest accounts' files (`businesses.storage_bytes`).
6. Accounts near 4 GB (the app refuses uploads at the limit): tell them to delete old files or ask for more.
7. Automatic jobs ran: access-log upload, erasing of files deleted over 30 days ago, trial and retention emails (worker log).

## 6 · How the measurements were made (repeatable)

- Empty database with the real structure (`npm run migrate` on a new database), then `sizing.sql` loaded 30 businesses with the volumes in 2.1
  (477,000 rows in 39 seconds), `VACUUM ANALYZE`, then table and index sizes, a compressed dump, the data files compressed with zstd, and the
  log (WAL) written during the load and after 200 separate one-row updates. The throw-away database was deleted afterwards.
- Access-log line size and compression: 60,000 realistic lines built with the real field list of `accessLogLine`.
- Limits of the method: the test text is repetitive (compression is better than real life, so the plan uses 3 to 1, not the measured 6 to 12 to 1), and real
  usage per person is an assumption until the pilot shows it. **After the first month of the pilot, replace the assumptions with the real numbers** from
  the R2 dashboard, `backup.sh info`, and the access-log file sizes.

> **Update 30 Oct 2026:** the limit is now **20 GB per account** (was 4 GB). For 30 accounts that is 600 GB of files at most (1,200 GB with the backup copy). The "typical" and "heavy" numbers above do not change, because they depend on how much people upload, not on the limit. See [32](32-files-and-reports-price-599.md).
