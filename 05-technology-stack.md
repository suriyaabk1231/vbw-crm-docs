# 5. Technology stack

**In one line:** one language (TypeScript) everywhere, free or open-source tools wherever possible.

| Layer | Tool | Version | Why | Cost |
|---|---|---|---|---|
| Web app | Next.js + React + Tailwind CSS | Next 15.5, React 19 | Fast, widely used, easy to hire for | Free |
| Android app | React Native + Expo (Expo Router) | Expo SDK 53 | Shares language and logic with web; iOS possible later | Free |
| API | Node.js + NestJS | Node 22, Nest 10 | Clean modules in one service | Free |
| Database | PostgreSQL (self-managed) | 16 | Reliable; Row-Level Security keeps businesses apart | Free |
| Cache + job queue | Redis + BullMQ | Redis 7 | Fast dashboard, scheduled jobs | Free |
| Search | PostgreSQL full-text + trigram | — | No extra search server | Free |
| Validation | Zod | 3 | Readable error messages | Free |
| Login | Firebase Authentication | — | Proven, Google runs password security; free up to 50,000 users | Free tier |
| Push notifications | Firebase Cloud Messaging | — | Android reminders | Free |
| Email | Brevo (or Amazon SES) | — | Branded reset, invite, trial emails | Free tier |
| Files + backups | Cloudflare R2 | — | 10 GB free, no download fees | Free tier, then about ₹1.3/GB |
| Backups | pgBackRest | — | Continuous, encrypted database backups | Free |
| CDN, SSL, firewall | Cloudflare | — | Speed and protection, hides server IP | Free |
| Web server | Caddy | 2 | HTTPS in front of the apps | Free |
| Private link | WireGuard | — | Encrypted main ↔ standby connection | Free |
| Containers | Docker + Docker Compose | — | Same setup on any server | Free |
| Payments (phase 2) | Razorpay | — | Subscriptions + payment links | About 2% per payment |
| WhatsApp | wa.me click-to-chat links | — | No API, no verification, no message fees | Free |
| Excel | ExcelJS | 4 | Import and export | Free |
| Tests | Vitest; k6 for load tests | — | Unit tests; 300-user load test | Free |
| Code + CI | Git + GitHub Actions | — | Version history, automatic tests | Free tier |
| Error tracking / uptime | Sentry, UptimeRobot | — | Know about problems quickly | Free tier |

## Why not…

| Option | Why not (for now) |
|---|---|
| Microservices | Too complex for a small team; one well-organised service is enough |
| Managed database (e.g. DigitalOcean) | About ₹2,500+/month extra; we self-manage with backups and a standby instead |
| Supabase free | Free projects pause after 1 week of inactivity — not safe for paying customers |
| Oracle Cloud Always Free | Idle servers can be reclaimed; free allowance reportedly cut in 2026 |
| WhatsApp Business API (now) | Needs Meta business verification and per-message fees; added in a later phase |
| iOS app | Most target customers use Android; can be added later from the same code |
