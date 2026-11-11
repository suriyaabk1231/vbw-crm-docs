# 23 · Data safety and security: CIA + AAA review (updated 27 Oct 2026, late)

Questions from the owner:
1. With many clients on one system, can one client ever see another client's data?
2. If data is deleted by mistake (by a client, by us, a bug or a server problem), what can we do?
3. Check everything against the **CIA triad** (Confidentiality, Integrity, Availability) and **AAA** (Authentication,
   Authorization, Accounting): only the right person sees exactly what they should. Fix the gaps.

**Result:** 10 gaps found, **8 fixed and tested today**, 2 need your Cloudflare account or come after the pilot.
Everything was tested on the local copy (27 Oct 2026). All 30 automatic tests pass; web, API and mobile type-check;
the web production build passes.

---

## 1 · Summary: every gap found, and its status

| # | Area | Gap found | Status | How it was checked |
|---|---|---|---|---|
| G1 | Integrity / Availability | Deleted patients were kept 30 days but **could not be brought back** | ✅ **Fixed.** Settings → Data → *Recently deleted* lists them with who deleted them and when they will be erased; **Restore** (owner only) | Member deletes → owner restores → back in list; member refused (403); merged duplicates not offered |
| G2 | Integrity / Availability | A deleted **file was erased at once**, and files had **no backup copy** | ✅ **Fixed.** Deleting hides a file for 30 days (storage freed at once); owner can restore. Every uploaded file gets a second copy in the backups bucket; a nightly job copies any it missed; if an original goes missing, it is put back from the copy automatically. After 30 days both copies are erased | Deleted, restored, storage count right; original deleted in storage → next download repaired it, content identical; nightly backup job and 30-day erase job run |
| G3 | Availability | Backups could only roll back **all** clients together | ✅ **Fixed.** `infra/scripts/restore-business.sh`: restore ONE client to a moment in time, other clients untouched; saves a copy first so the restore can be undone; all-or-nothing | Damaged a clinic (patient erased, name changed, appointment removed) → restored → undone → restored; gym never changed; search still works. *The step that reads the backup at a past time (pgBackRest) needs the real R2 backups and is tested on the server (doc 21, Part 7)* |
| G4 | Availability / Integrity | Updates took **no backup first**; no quick way back | ✅ **Fixed.** `deploy.sh` records the running version, makes a backup point, deploys, and checks the app is healthy; `rollback.sh` goes back to the previous version in one command | Script checker clean; full run needs the server |
| G7 | Integrity | Stages, WhatsApp messages and files deleted **without "are you sure?"** | ✅ **Fixed.** Confirm step on all three, saying what can and cannot be undone | Clicked in the browser |
| S1 | **Authentication** | Joining a team checked the email matched, but **not that the person owns that email**. Anyone can create an email + password login with someone else's address; with the invite link they could join and see all the client's data | ✅ **Fixed.** Joining needs a **verified email** (Google logins are always verified). The invite page sends the verification link and continues after it is clicked | Unverified → refused; other email → refused; verified → joined; used invite → refused |
| S2 | **Accounting** | Not recorded: **full Excel export** (most sensitive), device log-outs, stage and message changes, file uploads; entries had no names | ✅ **Fixed.** All recorded, with names; Settings → Data → Activity log now reads e.g. "Meena Receptionist · deleted patient Ramesh Kumar" | Each action done, each entry seen in the log |
| S3 | **Accounting / Integrity** | The app's database login could **change or delete audit entries** (a bug or an attacker could wipe the trail) | ✅ **Fixed.** Audit log is append-only for the app (migration 007) | App login tried to delete and update entries → "permission denied" |
| S4 | **Confidentiality** | Website lacked 2 browser protections | ✅ **Fixed.** HSTS (always HTTPS) and Permissions-Policy (no camera, microphone, location, payment, USB access) | Web build passes |
| G5 | Availability | The server holds keys that **can delete backups** | ⏳ **You, in Cloudflare:** turn on an R2 **bucket lock** on `vbw-crm-backups` (e.g. 14 days) so backups cannot be deleted even by someone who breaks into the server. Then run `backup.sh` once to check backups still work. Steps: section 4 | — |
| G6 | Availability | All backups are with one provider (Cloudflare) | ⏳ **After the pilot starts:** second backup copy on the standby server (pgBackRest supports two copies) | — |

Found during testing and fixed at once (not separate gaps): the restore script first failed on a computed database
column (it failed safely: nothing changed), and once wrote its copy outside the protected folder; both fixed, and any
`restores/` folder is now excluded from git because it holds customer data.

---

## 2 · CIA + AAA: what protects the data now

### Authentication: is this really the person?
| Control | Where |
|---|---|
| Firebase checks every login token on every request | `apps/api/src/common/auth.ts` |
| Email + password (8+ characters) or Google; no passwords stored by us | Firebase |
| **Verified email required to join a team** (S1) | `account.module.ts` → `acceptInvite` |
| 1 phone + 1 browser per person; logging in elsewhere logs the old device out | `devices` table |
| Removed team members are refused at once | `users.removed_at` |
| Test login refused on any server: the API will not start in production with it on, and it only answers this computer | `config.ts`, `auth.ts` |
| **Not yet:** two-step login (OTP or authenticator app) for owners | Later; Firebase needs its paid Identity Platform upgrade for it (check the price first) |
| **Not yet:** automatic log-out after hours of not using the app (shared clinic computers) | Later |

### Authorization: does this person have the right?
| Control | Where |
|---|---|
| Every client's rows are hidden from every other client **by the database itself** (row-level security on 14 tables) | migrations 001, 003 |
| The app's database login cannot switch that off (`NOBYPASSRLS`) | `infra/postgres/init/01-roles.sh` |
| The business is taken from the verified login, never from what the browser sends | `auth.ts` |
| **Owner only:** settings, stages, messages, invites, removing people, export, activity log, delete account, billing, **Recently deleted + Restore** | `@OwnerOnly()` on 20 routes (all routes reviewed) |
| A member can only log out their own devices | `revokeDevice` |
| Files: stored under the client's ID; download links signed and valid 5 minutes | `files.module.ts` |
| Tested: gym → clinic (read, edit, delete, merge, note, file download/delete, appointment, search, smuggled ID) all blocked; member → owner actions all blocked | section 3 |

### Accounting: who did what, and when?
| Control | Where |
|---|---|
| **Activity log** (owner sees it): sign-up, invites, joins, removals, patient add/change/delete/restore/merge/import, file upload/delete/restore, stages, messages, **export**, device log-outs, account deletion, business restore | `audit_log` |
| **Append-only** for the app (S3) | migration 007 |
| **Access log**: every request with time, user, IP and page (no search text, so no patient names), kept 1 year (DPDP) | `common/access-log.ts` |
| History on each patient's page | `activities` |

### Confidentiality
HTTPS everywhere (Cloudflare → server with an origin certificate); the server accepts web traffic only from Cloudflare;
database reachable only over the private server link; Redis not exposed; backups encrypted (AES-256); files private with
5-minute signed links; secrets in a `chmod 600` settings file; API and website security headers (S4 added two).

### Integrity
Database refuses rows for the wrong business; input checked on every request; file size checked after upload; deletes are
reversible for 30 days (G1, G2); confirm step before deletes (G7); audit cannot be altered (S3).

### Availability
Continuous encrypted database backup (≤ 5 minutes of loss, ~4 weeks back); weekly server snapshot; standby server; second
copy of every file (G2); single-client restore (G3); backup before every update + rollback (G4); monthly restore test;
rate limits; Cloudflare DDoS protection; uptime alert (doc 21, Part 9).

---

## 3 · Test results (local copy, 27 Oct 2026)

| Test | Result |
|---|---|
| Gym owner reads / edits / deletes / merges / notes a clinic patient | 404 every time |
| Gym owner downloads or deletes a clinic file; cancels a clinic appointment | 404 |
| Gym creates a record with the clinic's business ID added | saved under the gym |
| App database login, no business set | sees 0 rows; inserting into another business refused |
| Clinic member: export, delete stage, delete account, change settings, see/restore deleted items | 403 every time |
| Join a team with an unverified email / someone else's email / a used invite | refused (403 / 403 / 404) |
| Member deletes a patient and a file → owner restores both | works; storage count correct |
| File original lost in storage → download | repaired from the backup copy; content identical |
| Nightly jobs: second copies; erase after 30 days | copied; both copies erased after 30 days |
| One-client restore, undo, re-apply | correct; the other client unchanged |
| App database login tries to delete / change audit entries | permission denied |
| Automatic tests | 30 / 30 pass |

---

## 4 · What you need to do (G5)

1. Cloudflare dashboard → **R2** → `vbw-crm-backups` → **Settings** → **Bucket lock rules** → **Add rule**:
   all objects (no prefix), **14 days**. (Source: <https://developers.cloudflare.com/r2/buckets/bucket-locks/>)
2. On the server, run `infra/scripts/backup.sh full`, then `backup.sh info`: backups must still succeed.
   pgBackRest deletes backups older than its 4-week retention, which is longer than the 14-day lock, so this should not
   clash. Check the first weekly clean-up in the backup log anyway.
3. Do **not** lock `vbw-crm-files`: deleted customer files must be erasable after 30 days (privacy law).

---

## 5 · What to do when something goes wrong

| What happened | What to do |
|---|---|
| Client deleted a patient or file (within 30 days) | Owner: Settings → Data → Recently deleted → **Restore** |
| Client says their data is wrong since a date (up to ~4 weeks) | `restore-business.sh snapshot <business-id> "<time>"`, check counts with the client, then `apply … --yes` |
| Our update broke the app | `rollback.sh`; if data was damaged, restore to the "restore point before deploy" time in `infra/deploys/history` |
| A file is missing in storage | Opening it repairs it from the second copy automatically |
| Main server dies | Promote the standby (5–15 min) or restore from backup (*estimate* 1–2 h) |
| Owner deleted the whole account | Cancel within 7 days (Settings → Data) |

### Rules for the team
1. Never change the live database by hand; use the scripts.
2. Monthly restore test; write the result in doc 10.
3. `BACKUP_ENCRYPTION_PASSWORD` in two password managers.
4. Note every delete or restore done for a client (who asked, when, what).
5. Restore copies (`infra/restores/`) contain customer data: delete them once the client confirms.

---

## Sources
- R2 bucket locks: <https://developers.cloudflare.com/r2/buckets/bucket-locks/>
- Code: migrations 001, 003, 006 (`006_recovery.sql`), 007 (`007_audit_append_only.sql`); `apps/api/src/common/auth.ts`,
  `modules/files.module.ts`, `modules/contacts.module.ts`, `jobs/jobs.service.ts`; `infra/scripts/restore-business.sh`,
  `deploy.sh`, `rollback.sh`; `apps/web/next.config.mjs`
