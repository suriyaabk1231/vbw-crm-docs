# 21 · Step-by-step guide: run locally → push to GitHub → launch (as of 27 Oct 2026)

Work through this from top to bottom. Do not skip a step: each one is short, and later steps depend on earlier ones.
Steps marked **(you)** need your own login, money or decision. The detailed server commands are in
[12 — Launch checklist](12-launch-checklist.md); this guide puts every step in order and adds the local and GitHub parts.

State checked on 27 Oct 2026, 8:20 pm, on this computer:

| Check | Result |
|---|---|
| Database (PostgreSQL), Redis, file storage (s3mock) | running in Docker |
| API `http://localhost:4000/api/v1/health` | `{"ok":true}` |
| Web app | loads, no errors in the browser console |
| Background worker | running, 10 scheduled jobs |
| Database migrations | up to date |
| Tests | 30 of 30 pass (11 core + 19 API) |
| Type checks | web, API and mobile all pass |
| Production build of the web app | passes |
| Package lock file | consistent (`npm ci` will work on GitHub) |
| Secrets in git | none: only the empty `.env.example` templates are tracked |

---

## Part 1 · Run the app on your computer

You need: Docker Desktop, Node.js 20 or newer (this computer has 24.16), and the folder `C:\Users\Surya\Downloads\car web\vbw-crm`.

### 1.1 Start Docker Desktop
1. Open **Docker Desktop** from the Start menu and wait until it says "Engine running".
2. If it shows *"sailor-ingest.sock: The file cannot be accessed by the system"*, close it and run
   `C:\Users\Surya\Downloads\car web\fix-docker.ps1` (right-click → Run with PowerShell). Wait one minute.

### 1.2 Start the database, Redis and file storage
Open PowerShell in the `vbw-crm` folder and run:

```
npm run db:up
```

This starts 3 containers: `postgres` (port 5432), `redis` (6379) and `s3` (9090, the local stand-in for Cloudflare R2).
The first time, Docker downloads the images (a few minutes).

### 1.3 Build the shared code and update the database
```
npm run build:core
npm run migrate
```
`migrate` must end with `Migrations complete`. It runs from the API's build folder: after pulling new code with a new
migration file, run `npm run build -w @vbw/api` first (or start `dev:api` once), or the new file is skipped without warning.

### 1.4 Start the 3 programs (one PowerShell window each, leave them open)

| Window | Folder | Command | Ready when you see |
|---|---|---|---|
| 1 · API | `vbw-crm` | `npm run dev:api` | `Found 0 errors. Watching for file changes.` |
| 2 · Worker | `vbw-crm\apps\api` | `npm run start:worker` | `VBW CRM worker running` |
| 3 · Web | `vbw-crm` | `npm run dev:web` | `Ready` and `http://localhost:3000` |

Start the worker **after** the API says "Found 0 errors" (the worker runs the API's compiled files).
Do not run `npm run dev:worker` at the same time as `dev:api`: both would delete each other's build folder.

### 1.5 Check it works
1. Open `http://localhost:4000/api/v1/health` → must show `{"ok":true}`.
2. Open `http://localhost:3000` → the VBW CRM home page.

### 1.6 Log in (local test login, no password)
The local copy uses a test login instead of Firebase. In the browser on `http://localhost:3000`, press F12 → Console, paste one line and press Enter:

| Account | Line to paste |
|---|---|
| Clinic owner (has test patients) | `localStorage.setItem('vbw.devUser', JSON.stringify({uid:'qa-clinic-1027', email:'qa-clinic@test.local'}))` |
| Clinic member (read-only settings) | `localStorage.setItem('vbw.devUser', JSON.stringify({uid:'qa-member-1027', email:'qa-member@test.local'}))` |
| Gym owner | `localStorage.setItem('vbw.devUser', JSON.stringify({uid:'qa-gym-1027', email:'qa-gym-1027@test.local'}))` |
| Education owner | `localStorage.setItem('vbw.devUser', JSON.stringify({uid:'qa-education-1027', email:'qa-education-1027@test.local'}))` |

Then go to `http://localhost:3000/app`. Each person may use 1 browser: if you get "You were logged out because this account was
used on another device", paste the line again and reload.

To try a brand-new account: paste a line with a new uid and email (for example `uid:'me-1'`), then open `/signup`.

### 1.7 Stop everything
1. Press **Ctrl+C** in each of the 3 PowerShell windows.
2. `npm run db:down` (data is kept; it comes back with `npm run db:up`).

### 1.8 Problems and fixes

| What you see | Fix |
|---|---|
| `EADDRINUSE ... 4000` | An old API is still running. Close the old window, or restart the computer. |
| Web page stuck or blank after `npm run build` | Never build while `dev:web` runs. Stop it, delete `apps/web/.next`, start `dev:web` again. |
| "File storage is not configured" | `npm run db:up` was not run, or `apps/api/.env` lost the `R2_ENDPOINT=http://localhost:9090` lines. |
| Docker will not start | Step 1.1, the fix script. |

---

## Part 2 · Push to GitHub (you)

There are 2 git folders: `vbw-crm` (code, 12 commits, branch `main`) and `vbw-crm-docs` (documents, branch `main`).
Neither has a GitHub address yet.

### 2.1 Save the last documents in the docs folder
In PowerShell, in `vbw-crm-docs`:
```
git add -A
git commit -m "Handover, step-by-step guide, cloud plan, 100-day pilot (docs 20-22)"
```

### 2.2 Create the 2 repositories on GitHub
1. Log in at github.com (your account: suriyaabk11-max).
2. Top right **+** → **New repository** → name `vbw-crm` → choose **Private** → do **not** tick "Add a README", ".gitignore" or "license" → **Create repository**.
3. Do the same for `vbw-crm-docs` (also **Private**).

### 2.3 Connect and push the code
In PowerShell, in `vbw-crm`. First save the 27 Oct changes (100-day trial in the Terms draft and `.env.example`, swap in `harden-server.sh`):
```
git add -A
git commit -m "100-day pilot trial; swap memory in server hardening"
git remote add origin https://github.com/suriyaabk11-max/vbw-crm.git
git push -u origin main
```
The first push opens a GitHub sign-in window. Sign in in the browser; Windows remembers it.

### 2.4 Connect and push the documents
In `vbw-crm-docs`:
```
git remote add origin https://github.com/suriyaabk11-max/vbw-crm-docs.git
git push -u origin main
```

### 2.5 Check the automatic test on GitHub
1. Open the `vbw-crm` repository → **Actions** tab.
2. The run called **CI** starts by itself. It runs the tests, the full build, the database migrations, a check that one business
   cannot see another business's data, and the mobile app type check. It takes about 5 minutes.
3. It must show a **green tick**. All of these steps passed on this computer on 27 Oct 2026. If it shows a red cross, open it,
   copy the red error lines, and send them to Claude.

### 2.6 From now on, after every change
```
git add -A
git commit -m "What changed, in one line"
git push
```

---

## Part 3 · Accounts and keys (you)

### 3.1 Domain and Cloudflare
1. Buy a domain if you do not have one.
2. Create a Cloudflare account → **Add site** → your domain → Free plan.
3. At the shop where you bought the domain, change the nameservers to the 2 that Cloudflare shows. Wait until Cloudflare says "Active" (up to 24 hours).
4. Decide the 2 addresses: `app.<domain>` (web app) and `api.<domain>` (API).
5. SSL/TLS → set mode to **Full (strict)**.
6. SSL/TLS → Origin Server → **Create Certificate** for `app.<domain>` and `api.<domain>`. Save the certificate and the private key in 2 files on your computer (needed in step 5.6). The key is shown only once.

### 3.2 Cloudflare R2 (file storage)
1. R2 → create bucket `vbw-crm-files` (customer files).
2. Create bucket `vbw-crm-backups` (database backups and access logs).
3. R2 → Manage API tokens → **Create token** → permission **Object Read & Write**, only on these 2 buckets. Copy the Account ID, Access Key ID and Secret Access Key into a password manager.
4. `vbw-crm-files` → Settings → **CORS**: allow `PUT` and `GET` from `https://app.<domain>`.

### 3.3 Firebase (login)
1. console.firebase.google.com → **Create project**.
2. Authentication → Sign-in method → turn on **Email/Password** and **Google**.
3. Authentication → Settings → Authorised domains → add `app.<domain>`.
4. Project settings → Your apps → **Add app → Web**. Copy the 4 values: API key, auth domain, project ID, app ID.
5. Project settings → Service accounts → **Generate new private key**. Keep the JSON file private: never put it in git.

### 3.4 Brevo (email)
1. Create a Brevo account.
2. Senders & domains → add your domain → add the **SPF, DKIM and DMARC** records it shows into Cloudflare DNS. Wait until Brevo shows them as verified.
3. Add the sender `no-reply@<domain>`.
4. SMTP & API → **create an API key**. Copy it into your password manager.

### 3.5 Support details
1. Choose the WhatsApp number customers will message to activate their plan. Write it as digits only with country code, for example `9198XXXXXXXX`.
2. Choose a support email address.
3. Decide the **Grievance Officer** (name, email, phone). The DPDP Act 2023 needs one; it is shown in the Privacy Policy.

### 3.6 Lawyer
1. Send [17 — Terms, Privacy and DPA draft](17-terms-privacy-draft.md) to a lawyer.
2. Give them: legal entity name, registered address, the Grievance Officer details, and the open questions in doc 17.
3. Ask them in particular about the 90-day deletion of unpaid accounts (doc 17, Q7). Until they confirm, it stays off.

### 3.7 Check hosting prices (you)
The prices in [06 — Hosting and cost](06-hosting-and-cost.md) came from third-party sources. Check them on the Hetzner and Contabo
websites today, in INR including GST, before buying in Part 5.

---

## Part 4 · Test the real keys on your computer first

> **27 Oct 2026: you chose to go straight to the cloud, so skip this Part.** The same checks are done on the live server
> instead: [22 — Cloud plan](22-cloud-plan.md), section 7, step 8.

Put the real keys in the local files **only for this test**. Both files are ignored by git, so they cannot be pushed by accident.

1. `apps/api/.env`:
   - `FIREBASE_SERVICE_ACCOUNT_BASE64` = the Firebase JSON file from 3.3 step 5, converted to base64
     (PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\path\to\file.json"))`)
   - `BREVO_API_KEY`, `EMAIL_FROM_ADDRESS=no-reply@<domain>`
   - `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET=vbw-crm-files`
   - **delete** the 3 local storage lines `R2_ENDPOINT=...`, `R2_ACCESS_KEY_ID=dev`, `R2_SECRET_ACCESS_KEY=dev` (the real values replace them)
   - `AUTH_DEV_BYPASS=false`
2. `apps/web/.env.local`: the 4 `NEXT_PUBLIC_FIREBASE_*` values, `NEXT_PUBLIC_SUPPORT_WHATSAPP`, and `NEXT_PUBLIC_DEV_AUTH=false`.
3. For this test only: add `http://localhost:3000` to the R2 bucket CORS (3.2 step 4) and to Firebase authorised domains (3.3 step 3).
4. Restart the API, worker and web (Part 1, steps 1.4 and 1.7).
5. Check each of these:
   - [ ] Sign up with a real email and password → you land on the setup screen.
   - [ ] Log out → **Forgot password** → the email arrives in the inbox (not spam) → the link sets a new password.
   - [ ] Log in with **Google**.
   - [ ] Upload a PDF to a patient → download it → delete it. Settings → Business → Storage goes up, then back down.
   - [ ] Invite a second email → the invite email arrives → accept it in another browser → the owner sees 2 people.
   - [ ] Log in as the same person in a second browser → the first browser is logged out.
   - [ ] Settings → Plan & billing → **Message us on WhatsApp** opens a chat with your number.
6. Remove `http://localhost:3000` from R2 CORS and from Firebase again.
7. Put the local values back in `apps/api/.env` and `apps/web/.env.local` if you want to keep developing locally with the test login.

---

## Part 5 · Servers

Full commands are in [12 — Launch checklist](12-launch-checklist.md), sections C–E.

1. **(you)** Buy the main server (DigitalOcean, Bangalore, 2 vCPU / 4 GB, weekly backups on) and the standby (Contabo, Mumbai), as in [22 — Cloud plan](22-cloud-plan.md). Hetzner raised its prices in 2026, so doc 06 is out of date. Add your SSH key when ordering.
2. Log in to each with your SSH key and check it works **before** hardening (hardening turns off password login).
3. On each server: `git clone` the `vbw-crm` repository into `/opt/vbw-crm`.
4. On each server, as root: `/opt/vbw-crm/infra/scripts/harden-server.sh`. Then check from your computer: `ssh` still works, and `http://<server IP>` does **not** load.
5. WireGuard private link: follow `infra/wireguard/README.md` (main `10.8.0.1`, standby `10.8.0.2`). Check `ping 10.8.0.2` from the main server.
6. Install Docker Engine and the Compose plugin on both servers (Docker's official Ubuntu instructions).
7. On the main server: `cp infra/.env.example infra/.env` and fill in every value:
   - `APP_DOMAIN`, `API_DOMAIN`
   - `TRIAL_DAYS=100` for pilots (already the value in `infra/.env.example`). `infra/docker-compose.yml` passes the same value to the
     website, so the home page and sign-up page say "100 days" too. 100 is also the default everywhere, including on your computer.
   - `SUPPORT_WHATSAPP`, `SUPPORT_EMAIL`
   - `GRIEVANCE_NAME`, `GRIEVANCE_EMAIL`, `GRIEVANCE_PHONE`
   - `LAPSED_ACCOUNT_ERASURE=false` (until the lawyer confirms, 3.6)
   - `R2_BACKUP_BUCKET=vbw-crm-backups`
   - every password: generate each with `openssl rand -base64 32` and save a copy in your password manager.
     **If `BACKUP_ENCRYPTION_PASSWORD` is lost, the backups can never be restored.**
   - Firebase, Brevo and R2 values from Part 3. Leave `R2_ENDPOINT` **empty** on servers.
   - `ONLINE_PAYMENTS=false`; leave the Razorpay values empty
8. `chmod 600 infra/.env`
9. Save the certificate from 3.1 step 6 as `infra/secrets/cloudflare-origin.pem` and the key as `infra/secrets/cloudflare-origin.key`; `chmod 600` the key.
10. Cloudflare DNS: add `A` records `app` and `api` → main server IP, orange cloud (proxied) **on**.

---

## Part 6 · Start and check the live app

1. `cd /opt/vbw-crm && docker compose -f infra/docker-compose.yml up -d --build`
2. The `migrate` service log ends with `Migrations complete` (all files 001–005).
3. `https://api.<domain>/api/v1/health` shows `{"ok":true}`.
4. `https://app.<domain>` loads with a padlock.
5. The API refuses to start if a setting is unsafe (test login on, no Firebase key, an `http://` address). Read its log if it does not start.
6. Repeat every check from Part 4 step 5 on the real addresses.
7. Check the test login is really off: a request to `https://api.<domain>/api/v1/me` with header `x-dev-uid: test` must be refused.

---

## Part 7 · Backups and standby

1. First backup: `infra/scripts/backup.sh init`, then `infra/scripts/backup.sh full`.
2. Add the 2 cron lines from the top of `infra/scripts/backup.sh` (weekly full, daily differential).
3. **Restore test:** `infra/scripts/restore-test.sh`. The row counts must match the live database. Write the date in [10 — Operations](10-operations.md).
4. Standby: `REPLICATION_PASSWORD=... infra/standby/setup-replica.sh` on the standby server. The script prints whether replication runs.
5. **Switch-over practice** before customers have data: follow `infra/standby/promote.sh`, point DNS at the standby, check the app works,
   then rebuild the old main as the new standby. Write down how long it took.

---

## Part 8 · Load test (on a staging copy, never on the live app)

1. Set up a staging copy (the standby before it becomes a replica, or a temporary server) with
   `NODE_ENV=development`, `AUTH_DEV_BYPASS=true`, `AUTH_DEV_BYPASS_REMOTE=true`.
2. `k6 run -e API=https://<staging-api>/api/v1 infra/load-test/k6.js`
3. Pass if 95% of requests take under 300 ms and under 1% fail (300 people at once).
4. Turn both test-login settings off again, or delete the staging server.

---

## Part 9 · Before the first pilot customer

1. **(you)** Lawyer has approved the legal texts. Put them in `vbw-crm/apps/web/content/legal/` (terms, privacy, dpa, subprocessors),
   change `LEGAL_VERSION` in `packages/core`, commit, push, and redeploy (Part 6 step 1).
2. Fill in the 2 breach-procedure deciders in [10 — Operations](10-operations.md).
3. Test plan activation with your own test account:
   `docker compose -f infra/docker-compose.yml run --rm migrate node dist/db/activate.js --email <owner> --months 1 --note "UPI ref"`
4. **(you)** Real phones: an Android phone (Chrome) and an iPhone (Safari). Sign up, setup, add a patient, book an appointment,
   open the **More** menu (Tasks, Settings, Log out). Screens were checked at phone width on 27 Oct 2026, but never on a real phone.
5. Delete every test account you created in Parts 4 and 6.
6. Free uptime monitor on `https://api.<domain>/api/v1/health` that alerts your phone.

### Small fixes still open in the code (Claude can do these; none blocks launch)
- [ ] Error messages show internal field names, e.g. "sizeBytes: Files can be up to 25 MB".
- [ ] File delete uses the browser's plain "Delete this file?" box instead of the app's own confirm, and shows no "Deleted" message.
- [ ] Setup says "Invite sent" even when no email can go out (no Brevo key). Fine once Brevo is connected.
- [ ] Deleting a stage or a WhatsApp message does not ask "are you sure?".
- [ ] A few old style classes remain on the login pages.
- [ ] Accessibility pass (colour contrast of teal on dark, focus rings, screen-reader labels) and a security review.

---

## Part 10 · Sales and pilot launch (you)

1. Fill [16 — Target list](16-target-list-template.csv) with 100 businesses (hospitals, gyms, education centres, retail shops, real estate agents).
2. Call with [13 — Call script](13-call-script.md) (English + Tamil). Target: 30 pilots, 10 per industry.
3. Record the 2-minute demo video from [14](14-demo-video-script.md) (Tamil + English).
4. Design the brochure from [15](15-brochure-copy.md).
5. Pilot launch: set up each pilot yourself and import their Excel (Patients → Import Excel).
6. Pilots get 100 days free, then ₹599 + GST a month, locked for life ([09](09-go-to-market.md)).
7. When a pilot pays by UPI: run the activation command (Part 9 step 3) with their email and the UPI reference.

## Part 11 · Every week and month after launch

- Every week: call the pilots and collect feedback.
- Every 2 weeks: fix the top 3 complaints.
- Every day in the first month: check errors and speed.
- Every month: run `restore-test.sh` and write the result in doc 10.

---

## Part 12 · After phase 1

| Phase | Steps |
|---|---|
| **2 · Online payments** | Razorpay KYC → create the ₹599/month plan → add `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, `RAZORPAY_WEBHOOK_SECRET`, `RAZORPAY_PLAN_ID` → set `ONLINE_PAYMENTS=true` → add the webhook `https://api.<domain>/api/v1/webhooks/razorpay` (events `payment_link.paid`, `subscription.*`) → redeploy. The buttons appear by themselves. |
| **3 · Android app** | Restyle and test `apps/mobile` against the new backend → test on 2–3 real phones including a low-end one on slow internet → add Firebase `google-services.json` and the real API address in `app.json` → Google Play developer account (₹2,100) → privacy policy URL, store listing → build with EAS → submit for review. No payments inside the Android app. |
| **4 · Growth** | WhatsApp Business API inbox, IndiaMART/JustDial lead capture, automation rules, Indian languages, more industries (Real Estate first). Not built yet. |
