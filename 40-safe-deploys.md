# 40 — Safe deploys with no downtime

*Written 5 Nov 2026. Replaces the deploy steps in doc 10 / 21 / 23 (G4); `deploy.sh` and `rollback.sh` are run the same way as before.*

## 1. What changed and why

| Before | Now | Why it matters |
|---|---|---|
| The server **built** the new version itself (`docker compose build`) while customers were using it | **GitHub Actions builds** the images after every test passes and stores them in GitHub's registry (ghcr.io). The server only **downloads** them | Building uses most of the CPU and memory on a small server and could crash halfway. Code that fails a test now never gets an image, so it **cannot** be deployed |
| **One** API and **one** web container. Every deploy restarted them: 10–60 s of errors for everyone | **Two copies** of each. A deploy starts the new copies next to the old ones, waits until they answer their health check, then stops the old ones | **No outage** during a deploy. If one copy crashes, the other keeps serving |
| A broken new version was already live before the health check failed | If a new copy never becomes healthy, it is removed and the **old version keeps running** | A bad deploy no longer takes the site down |
| Caddy sent everything to one container | Caddy spreads requests over both copies and retries on another copy for up to 10 s if one is stopping or broken | Customers do not see errors while copies are swapped |
| Health checks used `localhost`, which never reached the app (section 4) | Health checks use `127.0.0.1` | The API health check never passed before, so deploy.sh always reported an error |
| Two deploys at the same time could mix versions | A lock allows one deploy or rollback at a time | — |
| After a rollback, the next `deploy.sh` failed (detached checkout) | `deploy.sh` switches back to `main` first | — |

Files: `.github/workflows/ci.yml` (job `images`), `infra/docker-compose.yml`, `infra/Caddyfile`, `infra/scripts/deploy.sh`, `infra/scripts/rollback.sh`, `infra/scripts/lib-release.sh`, `infra/.env.example`.

## 2. One-time setup (your side, about 15 minutes)

1. **GitHub repository variables** (Settings → Secrets and variables → Actions → **Variables** tab). These are built into the web app; none of them is secret. Same values as in the server's `infra/.env`:
   `API_DOMAIN`, `NEXT_PUBLIC_FIREBASE_API_KEY`, `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`, `NEXT_PUBLIC_FIREBASE_PROJECT_ID`, `NEXT_PUBLIC_FIREBASE_APP_ID`, `TRIAL_DAYS`, `SUPPORT_WHATSAPP`, `ONLINE_PAYMENTS`, `SUPPORT_EMAIL`, `GRIEVANCE_NAME`, `GRIEVANCE_EMAIL`, `GRIEVANCE_PHONE`.
   Until `API_DOMAIN` is set, the `images` job is skipped and everything works as before.
2. **Push to `main`** and check the Actions tab: the `images` job should be green. The images appear under your GitHub profile → Packages (`vbw-crm-api`, `vbw-crm-web`). They are private.
3. **Let the server download them** (on the server, once). Create a GitHub *classic* token with only `read:packages`, then:
   ```bash
   docker login ghcr.io -u suriyaabk11-max
   ```
   (paste the token as the password).
4. In the server's `infra/.env` add:
   ```
   IMAGE_REGISTRY=ghcr.io/suriyaabk11-max/vbw-crm
   ```
5. Run `infra/scripts/deploy.sh` as usual.

**Staging:** leave `IMAGE_REGISTRY` empty there. Staging then builds its own images (its web app needs the staging API address built in), with the same no-downtime swap.

## 3. How a deploy runs now

1. Backup point (as before).
2. `git pull` (switching back to `main` after a rollback).
3. Get the images for this commit. If GitHub Actions is still running, it waits up to 20 minutes. **If the tests failed, it stops here and nothing changes on the server.**
4. Postgres, Redis and Caddy keep running (Caddy restarts for a second only if the Caddyfile changed).
5. Database migrations.
6. API: 2 new copies start → health check → the 2 old copies finish their requests and stop.
7. Web: the same.
8. Worker: replaced directly (there must only be one; scheduled jobs run a few seconds later).

`rollback.sh` does the same without migrations, using the images that are still on the server.

## 4. What was tested and what was not

Tested on 5 Nov 2026 on the development PC (Docker 29.7). The setup used the real `lib-release.sh` swap and the same Caddy settings as `infra/Caddyfile`, with a stand-in API: 3 s start-up, 200 ms per request, clean stop. **6 clients sent GET and POST requests the whole time.**

| Test | Result |
|---|---|
| 5 swaps in a row (v13 → v17) | **0 failed of 1,264 requests**, about 13 s per swap |
| Broken new version (crashes on start) | Refused in 8 s, **0 failed of 297**, old version kept serving, broken copies removed |
| Good version right after the broken one | **0 failed of 257**, swapped normally |

The test found **three bugs, all fixed** before the results above:

1. **Health checks always failed (also in the old setup).** Inside the containers, `localhost` means the IPv6 address `::1`, but the API and web app listen only on IPv4. So the old `api` health check in `docker-compose.yml`, and the old `deploy.sh` check, could never pass. All checks now use `127.0.0.1`.
2. **About 1 request in 1,000 failed during a swap.** Caddy reused open connections, so the old copies never went idle and were force-stopped after 30 s. Now Caddy opens a new connection for each request (`keepalive off`), so a stopping copy refuses new requests and Caddy retries on another copy.
3. **Crashed copies were left behind** and restarted by the next deploy. They are now found and removed, and a broken copy is stopped gently, not killed.

**Not tested yet:**
- The GitHub Actions `images` job (needs the variables in step 2.1 and a push).
- A real deploy on the server with the real API and web app.
- Run the first deploy at a quiet hour and watch `docker compose ps`.

## 5. Limits to know

- **Memory:** 2 API + 2 web copies use about 300–400 MB more than before *(estimate)*; for about a minute during a deploy there are 4 of each. On the 2 vCPU / 4 GB server, watch memory after the first deploy (doc 34, section 5: act above 80%).
- **Database connections:** each API copy may open up to 25. 2 copies + worker = 75 of the 100 allowed; during the swap the old copies close theirs as they stop. If `/health/metrics` ever shows connections near 100, add PgBouncer (doc 34).
- **`/health/metrics` is per copy:** each request lands on one of the two API copies, so the numbers describe that copy only.
- **Changing Postgres or Redis settings** still restarts them (a short outage). Do that at night.
