# 12 — Launch checklist (phase 1: web app, manual UPI payments)

Work through this from top to bottom. Tick each box as you go. Every step names the exact file,
setting or command. Steps marked **(you)** need your own logins or money. Everything else can be done
by whoever is setting up the servers.

Code: `vbw-crm/` (latest commits: security hardening and phone-layout fixes, 24 Oct 2026).
Costs for each service are in [06 — Hosting and monthly cost](06-hosting-and-cost.md).

---

## A. Accounts and keys (you)

### A1. Domain and Cloudflare
- [ ] Create a Cloudflare account and add the domain. Change the domain's nameservers at the registrar to the two Cloudflare gives you.
- [ ] Decide the two addresses, for example `app.<domain>` (web app) and `api.<domain>` (API).
- [ ] SSL/TLS → mode **Full (strict)**.
- [ ] SSL/TLS → Origin Server → **Create Certificate** for `app.<domain>` and `api.<domain>`.
      Save the certificate as `origin.pem` and the private key as `origin.key`. Keep them private: they go on the server in step C5.
- [ ] R2 → create two buckets: `vbw-crm-files` (customer files) and `vbw-crm-backups` (database backups).
- [ ] R2 → Manage API tokens → create a token with **Object Read & Write** on those two buckets only.
      Note the Account ID, Access Key ID and Secret Access Key.
- [ ] R2 → `vbw-crm-files` → CORS: allow `PUT` and `GET` from `https://app.<domain>` (browsers upload files straight to R2).

### A2. Firebase (login)
- [ ] Create a Firebase project. Authentication → Sign-in method → turn on **Email/Password** and **Google**.
- [ ] Authentication → Settings → **Authorised domains**: add `app.<domain>`.
- [ ] Project settings → Your apps → add a **Web app**. Copy the 4 values (API key, auth domain, project ID, app ID).
- [ ] Project settings → Service accounts → **Generate new private key** (a JSON file).
      Turn it into one line of base64: `base64 -w0 service-account.json`. Delete the JSON file afterwards.

### A3. Brevo (email)
- [ ] Create a Brevo account. Senders & domains → add the domain and add the **SPF, DKIM and DMARC** records it shows in Cloudflare DNS.
      Without these, password-reset and invite emails land in spam.
- [ ] Add the sender `no-reply@<domain>`.
- [ ] SMTP & API → create an **API key**.

### A4. GitHub (code)
- [ ] Create 2 **private** repositories: `vbw-crm` and `vbw-crm-docs`.
- [ ] Push both: `git remote add origin <url>` then `git push -u origin main` (in each folder).
- [ ] In `vbw-crm`, check that the CI run (`.github/workflows/ci.yml`) is green. It runs the tests, the build, the migrations
      and the Row-Level Security check.

### A5. Support WhatsApp
- [ ] Choose the WhatsApp number customers will message to activate their plan. Format: country code + number, digits only (for example `9198XXXXXXXX`).

---

## B. Test the real keys on your computer first

Put the real keys in the local files **only to test**. Never commit them: both files are already in `.gitignore`.

| File | Variables to fill |
|---|---|
| `apps/api/.env` | `FIREBASE_SERVICE_ACCOUNT_BASE64`, `BREVO_API_KEY`, `EMAIL_FROM_ADDRESS`, `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`; set `AUTH_DEV_BYPASS=false` |
| `apps/web/.env.local` | the 4 `NEXT_PUBLIC_FIREBASE_*` values, `NEXT_PUBLIC_SUPPORT_WHATSAPP`; set `NEXT_PUBLIC_DEV_AUTH=false` |

For this local test, add `http://localhost:3000` to the R2 bucket CORS and to Firebase's authorised domains. Remove both after the test.

- [ ] Sign up with a real email and password → you land on setup.
- [ ] Log out → **Forgot password** → the email arrives (inbox, not spam) and the link sets a new password.
- [ ] Log in with **Google**.
- [ ] Upload a PDF to a contact → download it again → delete it. The storage figure in Settings goes up and back down.
- [ ] Invite a second email → the invite email arrives → accept it in another browser → the owner sees 2 people.
- [ ] Log in as the same person in a second browser → the first browser is logged out (1 browser per person).
- [ ] Settings → Plan & billing → **Message us on WhatsApp** opens a chat with your number.

---

## C. Servers

### C1. Buy (you)
- [ ] Check current prices again and buy the **main server (Hetzner, Singapore)** and the **standby (Contabo, Mumbai)**, sizes as in [06](06-hosting-and-cost.md).
      Choose Ubuntu LTS and add your SSH public key while ordering.

### C2. Harden both servers
- [ ] Log in with your SSH key and check it works **before** hardening (the script turns off password login).
- [ ] Copy the repository to `/opt/vbw-crm` (`git clone` from GitHub).
- [ ] Run as root on **each** server: `/opt/vbw-crm/infra/scripts/harden-server.sh`
      This sets up automatic security updates, SSH keys only, fail2ban, and a firewall that allows web traffic **only from Cloudflare**.
- [ ] Check from your computer: `ssh` still works. Opening `http://<server IP>` directly should **not** load.

### C3. WireGuard (private link main ↔ standby)
- [ ] Follow `infra/wireguard/README.md`. Main = `10.8.0.1`, standby = `10.8.0.2`.
- [ ] Check: `ping 10.8.0.2` from the main server works.

### C4. Docker
- [ ] Install Docker Engine + the Compose plugin on both servers (from Docker's official Ubuntu instructions).

### C5. Settings file and certificate (main server)
- [ ] `cp infra/.env.example infra/.env`, then fill it in:
  - `APP_DOMAIN`, `API_DOMAIN`
  - `TRIAL_DAYS=100` (pilot), `SUPPORT_WHATSAPP` and `SUPPORT_EMAIL`
  - **Grievance Officer** (shown in the footer and Privacy Policy): `GRIEVANCE_NAME`, `GRIEVANCE_EMAIL`, `GRIEVANCE_PHONE`.
    Until these are set, the Privacy Policy shows "[not set yet]".
  - `LAPSED_ACCOUNT_ERASURE=false` until the lawyer confirms the 90-day retention period ([17](17-terms-privacy-draft.md), Q7)
  - `R2_BACKUP_BUCKET` also receives the daily access logs (`access-logs/`, kept 1 year by the worker)
  - every password with `openssl rand -base64 32`. Keep a copy in a password manager.
    **Losing `BACKUP_ENCRYPTION_PASSWORD` means the backups cannot be restored.**
  - Firebase, Brevo and R2 values from section A
  - `ONLINE_PAYMENTS=false`; leave the Razorpay values empty
- [ ] `chmod 600 infra/.env`
- [ ] Save the certificate from step A1 as `infra/secrets/cloudflare-origin.pem` and the key as `infra/secrets/cloudflare-origin.key`
      (these exact names are what `infra/docker-compose.yml` gives to Caddy). `chmod 600` the key.

### C6. DNS
- [ ] In Cloudflare DNS, add `A` records `app` and `api` → main server IP, with the **orange cloud (proxied) on**.

---

## D. Start and check

- [ ] `cd /opt/vbw-crm && docker compose -f infra/docker-compose.yml up -d --build`
- [ ] Migrations ran (the `migrate` service). The log must end with `Migrations complete`, including `003_security_hardening.sql`.
- [ ] `https://api.<domain>/api/v1/health` shows `{"ok":true}`.
- [ ] `https://app.<domain>` loads with a padlock.
- [ ] The API **refuses to start** if a setting is unsafe (dev login on, no Firebase key, an `http://` address).
      If it keeps restarting, read `docker compose logs api` for the line starting `Unsafe production settings`.
- [ ] Repeat the checks in section B on the real addresses.
- [ ] Check the dev login is really off: send a request to `https://api.<domain>/api/v1/me` with header `x-dev-uid: test`.
      It must answer `NOT_LOGGED_IN`.

---

## E. Backups and standby

- [ ] First backup: `infra/scripts/backup.sh init`, then `infra/scripts/backup.sh full`.
- [ ] Add the two cron lines from the top of `infra/scripts/backup.sh` (weekly full, daily differential).
- [ ] **Restore test:** `infra/scripts/restore-test.sh`. The row counts must match the live database. Write the date in [10 — Operations](10-operations.md).
- [ ] Standby: `REPLICATION_PASSWORD=... infra/standby/setup-replica.sh` on the standby server.
      Check replication is running (the script prints the status).
- [ ] **Switch-over practice** (do this before customers have data, or on a copy):
      follow `infra/standby/promote.sh`, point DNS at the standby, check the app works, then rebuild the old main as the new standby.
      Write down how long it took.

---

## F. Load test (staging, never production)

- [ ] Set up a staging copy (the standby before it becomes a replica, or a temporary server) with
      `NODE_ENV=development`, `AUTH_DEV_BYPASS=true`, `AUTH_DEV_BYPASS_REMOTE=true`.
- [ ] `k6 run -e API=https://<staging-api>/api/v1 infra/load-test/k6.js`
- [ ] Pass if **95% of requests take under 300 ms** and **under 1% fail** (300 people at once).
- [ ] Turn both dev-login settings off again, or delete the staging server.

---

## G. Before the first pilot customer

- [ ] **(you)** Lawyer has approved the Terms of Service and Privacy Policy, starting from the draft in [17](17-terms-privacy-draft.md) (DPDP Act 2023; data stored in Singapore and Mumbai;
      service providers: Firebase, Brevo, Cloudflare, Hetzner, Contabo).
- [ ] Put the approved texts in `vbw-crm/apps/web/content/legal/` (terms, privacy, dpa, subprocessors) and change `LEGAL_VERSION`
      in `packages/core/src/constants.ts` (remove `-draft`). The pages, footer links and sign-up checkbox already exist.
- [ ] Fill in the two breach-procedure deciders in [10 — Operations](10-operations.md).
- [ ] Test the activation command on the server with your own test account:
      `docker compose -f infra/docker-compose.yml run --rm migrate node dist/db/activate.js --email <owner> --months 1 --note "UPI ref"`
- [ ] Phone check on a real Android phone (Chrome) and an iPhone (Safari): sign-up, setup, add a contact, book an appointment,
      the **More** menu (Tasks, Settings, Log out). Screens were checked at phone width on 24 Oct 2026, but not yet on real phones.
- [ ] Delete every test account you created during sections B and D.
- [ ] Uptime check: a free monitor on `https://api.<domain>/api/v1/health` that alerts your phone.

---

## H. Week of launch

- [ ] Onboard pilots: 100 days free, then ₹599 + GST a month, locked for life ([09 — Go-to-market](09-go-to-market.md)).
- [ ] Every week: collect feedback; every 2 weeks: fix the top 3 complaints.
- [ ] Every month: run `restore-test.sh` and note the result.

---

## Security already built in (for reference)

| Protection | Where |
|---|---|
| Each business sees only its own data (Row-Level Security) | `apps/api/src/db/migrations/001_init.sql` |
| Records cannot point at another business's records | `003_security_hardening.sql` |
| The activity log cannot be edited or deleted by the app | `003_security_hardening.sql` |
| User-typed names are escaped in emails | `escapeHtml` in `apps/api/src/common/services.ts` |
| Dev login works only on a developer's own computer, never in production | `apps/api/src/common/auth.ts`, `config.ts` |
| The API refuses to start with unsafe production settings | `apps/api/src/config.ts` |
| Rate limits (for example 5 sign-ups an hour per address) | `apps/api/src/common/auth.ts` |
| Razorpay webhooks checked by signature | `apps/api/src/common/services.ts` |
| Server accepts web traffic only from Cloudflare; SSH by key only | `infra/scripts/harden-server.sh` |
| Encrypted daily backups to R2, monthly restore test | `infra/scripts/backup.sh`, `restore-test.sh` |
