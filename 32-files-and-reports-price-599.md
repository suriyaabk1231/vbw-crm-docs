# 32 — Files and reports per customer, price ₹599, 20 GB storage

*Decided and built on 30 Oct 2026.*

## 1. What changed

| Item | Before | Now |
|---|---|---|
| Price | ₹399 per account per month + GST | **₹599 per account per month + GST** |
| Storage | 4 GB per account | **20 GB per account**, shared by both people |
| Files per customer | Web: one file at a time on the customer's page. Phone: camera photo only, no list of files | **Web and phone:** several files at once, list with size and date, open/download, delete |
| Attaching while adding a lead/customer | Not possible | **Possible on web and phone** — pick files in the "Add" form; they upload right after the person is saved |

Who can use it: **every person in the account** (Owner and Member). Only restoring a deleted file stays owner-only (Settings, 30 days).

Example: a hospital or dental clinic adds a new patient, attaches the X-ray scan and the lab report in the same form, and on the next visit opens them from the patient's page.

Allowed files (unchanged): PDF, JPG, PNG, WEBP, Excel (XLSX/XLS), CSV — up to 25 MB each.

## 2. Where it is in the code

| Place | File |
|---|---|
| Price and storage constants | `packages/core/src/constants.ts` (`PRICE_PAISE = 59_900`, `STORAGE_PER_ACCOUNT_BYTES = 20 GB`) |
| Web: "Attach files" in the Add/Edit form | `apps/web/src/components/app/dialogs.tsx` (`AttachFiles`, used by `ContactDialog`) |
| Web: "Files and reports" card | `apps/web/src/app/app/contacts/[id]/page.tsx` |
| Web: several uploads | `apps/web/src/lib/api.ts` (`uploadFiles`) |
| Phone: file picker, upload, file rows | `apps/mobile/src/files.tsx` (new; uses `expo-document-picker`) |
| Phone: "Add files" + "Files and reports" card | `apps/mobile/app/contact/[id].tsx` |
| Phone: "Attach files" while adding | `apps/mobile/app/contact/new.tsx` |
| Test: Owner and Member both attach and download | `apps/api/test/integration/files.test.ts` |

The API did not need new endpoints: files already belonged to a customer (`contactId`). The 20 GB limit is checked by the same code that checked 4 GB.

## 3. What it costs (Cloudflare R2)

Cloudflare R2 charges **$0.015 per GB-month** for Standard storage after **10 GB-month free**, with no download fees ([R2 pricing](https://developers.cloudflare.com/r2/pricing/)). Every file is stored twice (original + backup copy, doc 29).

| Case (30 pilot accounts) | Files | With backup copy | Cost per month |
|---|---|---|---|
| Typical (doc 29) | ~7 GB | ~16 GB | ≈ ₹8 |
| Heavy (doc 29) | ~45 GB | ~94 GB | ≈ ₹120 |
| Worst case: every account fills 20 GB | 600 GB | 1,200 GB | ≈ $18 ≈ **₹1,600** |

*Estimate:* rupee figures use about ₹88 per US dollar; check the rate on the day. The worst case is unlikely but is ~₹1,450 more than with 4 GB, so it fits inside the extra ₹200 per account (₹6,000 a month for 30 accounts).

## 4. Your side (not code)

1. **Razorpay:** when online payments start, create a **new ₹599/month plan** in the Razorpay dashboard and put its id in `RAZORPAY_PLAN_ID` (a plan's amount cannot be edited).
2. **Pilots already told ₹399:** the Terms (3.2) promise the pilot price "for as long as the account stays continuously paid". Anyone who already accepted a written ₹399 offer should keep it — decide this before sending new offers.
3. Sales material (call script, brochure, demo video script) now says ₹599 — re-record or reprint anything already made.
4. Phone app: run `npm install` once (new package `expo-document-picker`) and build a new APK.
