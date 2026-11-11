# 18 — Claude Design: steps and prompts for the VBW CRM front end

Goal: use Claude Design to choose **how the app looks**. Claude Code then builds that look into the real app
(`vbw-crm/apps/web`), connected to the existing backend, so every feature, the login and the security keep working.

The feature list below was taken from the code on 26 Oct 2026. If a screen is not listed here, the backend does not support it.
**Do not add features in Claude Design that are not in this list**; ask first, so the backend can be built for them.

---

## Steps

1. **Open Claude Design → choose the "UI mockups" template.** Leave Design system on **None**. Model: Opus 5.5.
2. **Send Prompt 1** (style + dashboard). It shows 2–3 styles of one screen.
3. **Pick one style.** Refine it by clicking an element and commenting (for example "bigger cards", "less text"). Stop when you like it.
4. **Send Prompt 2** (website and login screens), then **Prompt 3** (daily work screens), then **Prompt 4** (settings and special states),
   in the **same project**, so they keep the chosen style. Check each batch against the checklist at the end.
5. **Export:** for each screen, take a screenshot. If Claude Design lets you export code (the `</>` button), export it too.
   Save everything in `C:\Users\Surya\Downloads\car web\design\` (one folder per prompt).
6. **Tell Claude Code:** "The designs are in the design folder. Build them into the app."
   Claude Code will match every screen to the existing features, keep login and security, and test each screen at phone and desktop width.
7. **Review** the screenshots Claude Code sends back, and approve before anything is committed.

Tips: use only made-up names. Claude Design uses a lot of your allowance; if it runs low, stop after Prompt 1 and let Claude Code apply the style to the rest.

---

## Prompt 1 — style and dashboard (send first)

```text
I am designing the web app "VBW CRM": a simple CRM for small Indian businesses — clinics, gyms and coaching centres.
Users are clinic receptionists, doctors, gym owners and coaching-centre staff. Not technical. Mostly on a phone browser, sometimes a computer.
Price ₹599/month + GST for 2 people. Language: simple English.

Design ONLY the Dashboard (home) screen of a clinic, in 3 different visual styles side by side, each at phone width (375px)
and desktop width (1280px), so I can choose one style for the whole app.

Rules for every style:
- Main colour teal #0f766e. WhatsApp green #25D366 only on WhatsApp buttons. Red only for delete/danger.
- Big tap targets (at least 44px), readable 15–16px text, clear contrast, friendly and calm, no clutter.
- Phone: bottom navigation bar with 5 items: Home, Patients, Appointments, Follow-ups, More.
- Desktop: left sidebar with: Home, Patients, Appointments, Follow-ups, Pipeline, Tasks, Settings; business name at the top;
  user name and "Log out" at the bottom.
- Made-up data only.

Dashboard content:
- Greeting "Good morning, Priya" + "Here is your day."
- Two buttons: "+ Appointment" and "+ Patient".
- 4 number cards: Appointments today 6 · Follow-ups due (7 days) 4 · No-shows (7 days) 1 · Tasks today 3.
- "Today's appointments" list with a "Calendar" link: each row = time · patient name · appointment type · status badge (Booked/Done/No-show/Cancelled)
  and buttons WhatsApp, Done, No-show. Empty state: "No appointments today".
- "Tasks for today" list with a checkbox per task (title, due time, linked patient). Empty: "Nothing due 👍".
- "Follow-ups due in the next 7 days" with "See all": patient name, type, due date, amount in ₹, and a green "Remind on WhatsApp" button.
- An optional thin notice bar at the very top, e.g. "Your free trial ends in 3 days. Message us on WhatsApp to activate (₹599/month)".
```

---

## Prompt 2 — website, login and setup

```text
Use the style I chose. Design these screens at phone (375px) and desktop (1280px) width:

1. LANDING PAGE (public website)
   - Top bar: "VBW CRM" logo text, "Log in" button, "Start free trial" button.
   - Hero: "Follow up every patient, member and student — on time." Sub-text about appointments, renewals, fee reminders and WhatsApp
     in one place, on web and Android. Buttons "Start 100-day free trial" and green "Book a demo on WhatsApp".
     Small line: "Set up in 5 minutes · ₹599/month for 2 people".
   - 3 industry cards — Clinic ("Never miss a patient follow-up"), Gym & Fitness ("Stop losing members at renewal time"),
     Education Centre ("Every enquiry followed up, every fee collected on time") — each with 3 problems it solves:
     Clinic: patients forget follow-up visits, appointment book is on paper, no-shows are not tracked.
     Gym: memberships expire without renewal, trial visitors are never called back, fees chased on phone.
     Education: enquiries get lost in WhatsApp, demo classes are not followed up, fee instalments are paid late.
   - Features grid: Appointments & sessions, Renewals & fee reminders, WhatsApp in one tap, Morning summary, Excel import,
     Works on your phone, Your data is yours.
   - Price card: "One simple price" ₹599/month + GST, per account · 2 people · 20 GB storage shared; ✓ All features included,
     ✓ Works on computer and phone, ✓ WhatsApp reminders, ✓ Export your data any time; button "Start free trial".
   - FAQ (expandable): Is there a free trial? Who can use one account? Do I need the WhatsApp Business API? How do I pay?
     Can I take my data out?
   - Footer: Terms of Service · Privacy Policy · Data Processing Agreement · Sub-processors · "Privacy questions or complaints:
     Grievance Officer" · © VBW.

2. LOG IN: email, password, "Log in" button, "Log in with Google" button, "Forgot password?" link, "New here? Start free trial" link.
   Error message area (e.g. "Wrong email or password").
3. SIGN UP — step 1 "Start your free trial — 100 days free. No card needed": Your name, Business name (placeholder "e.g. Sri Sai Clinic"),
   Email, Password (8+ characters), a REQUIRED checkbox "I agree to the Terms of Service and Privacy Policy, and to the Data Processing
   Agreement for my business" (the three names are links), "Start free trial" button, "Sign up with Google", "Already have an account? Log in".
   Step 2 "Finish creating your account — Logged in as name@email.com": Your name, Business name, the same checkbox, "Create account".
4. FORGOT PASSWORD: email field, "Send reset link"; success message "If this email has an account, we sent a link."
5. SET NEW PASSWORD (from the email link): New password (8+), Type it again, "Save password"; states: "Checking link…",
   "This link is incomplete. Request a new reset link.", success "Password changed — You can now log in".
6. JOIN YOUR TEAM (invite link): "Join your team on VBW CRM — Use the email address the invite was sent to": Your name, Email, Password,
   the agree checkbox (Terms + Privacy only), "Join", toggle "I already have a password" / "I am new here".
7. SETUP WIZARD (after sign-up), 5 steps with a progress bar; on phone show "Step 2 of 5 · Your business" instead of 5 labels:
   Step 1 "What kind of business is it?" — 3 big choice cards: Clinic 🩺, Gym & Fitness 🏋️, Education Centre 🎓 (with their one-line pitch).
   Step 2 "About your business": Business name, Phone, City; Back / Next.
   Step 3 "Bring in your patients": "Import from Excel / CSV" button, "Skip for now".
   Step 4 "Invite your partner — Each account is for 2 people, for example you and your receptionist": email field + "Invite", Back / Skip.
   Step 5 "You are ready! 🎉" + "Go to my dashboard".
All public pages (1–6) show the same footer links as the landing page.
```

---

## Prompt 3 — daily work screens

```text
Same style. Phone (375px) and desktop (1280px). Clinic words shown; note that gyms say Members / Sessions / Renewals and
coaching centres say Students / Sessions / Fees — the design must fit longer or shorter words.

1. PATIENTS LIST: title "Patients (124)", buttons "Import Excel" and "+ Add patient", search "Search by name, phone or email",
   stage filter dropdown (All stages + each stage), list/table: Name, Phone, Stage badge, Tags. Long names must wrap.
   Empty state "No patients yet — Add them one by one or import your Excel list."; search empty "No matches". Pagination (50 per page).
2. ADD / EDIT PATIENT (bottom sheet on phone, dialog on desktop): Name *, Phone, Email, industry fields
   (clinic: Age, Gender, Doctor, Visit reason; gym: Plan, Trainer, Goal, Start date; coaching: Course, Batch, Parent's phone, Total fee),
   any custom fields the owner added, Tags (comma separated, e.g. "VIP, referral"), Stage; Cancel / Save.
   Duplicate warning: "Priya Lakshmi already has this phone or email" with "Open existing" and "Merge".
3. PATIENT PAGE: name, phone, email (long emails must wrap), Edit link; big "📞 Call" and green "WhatsApp" buttons; Stage dropdown;
   tags; "Delete" (red, with confirm). Sections:
   - Appointments (+ Book): date/time · type · status badge · "Remind".
   - Follow-ups (+ Add): type · due date · amount · status (Due/Paid/Renewed/Cancelled) · "Mark paid" (in the same step it sets the
     next due date, e.g. next month's fee).
   - Open tasks (+ Task).
   - Files: "Upload file (max 25 MB)" — PDF, images, Excel/CSV; each file: name, size, download, delete; storage note.
   - Notes: "Write a note…" box + Save.
   - History timeline: notes, calls with outcome, WhatsApp messages sent, stage changes, appointments, payments, files — with time and who.
4. AFTER A CALL (opens when coming back from a call): "How did the call go?" 4 big choices: Interested · Call later · Not interested ·
   No answer; "Remind me to call again" date-time; "Note (optional)"; Save.
5. SEND ON WHATSAPP (dialog): choose a ready message (e.g. Appointment reminder, Follow-up reminder, Missed appointment) with the
   text pre-filled with name/date/time/amount; "Or write your own" text box; green "Open WhatsApp" button
   (it opens the user's own WhatsApp — the app never sends by itself).
6. PIPELINE: horizontal columns, one per stage (clinic: Enquiry → Appointment booked → Visited → Follow-up due → Completed / Not coming),
   count per column, patient cards (name, phone) that can be dragged between columns; scrolls sideways on phone.
7. APPOINTMENTS (calendar): "+ Book appointment"; ← Today → date navigation with the date "Saturday, 24 October"; Day / Week toggle;
   day list: time · patient · type · duration · status badge; buttons Remind (green), Done, No-show, Cancel. Week view: 7 columns.
8. BOOK APPOINTMENT (dialog): For (patient search), Type (Consultation / Follow-up visit / Procedure), Date and time, Minutes, Notes; Book.
9. FOLLOW-UPS: "+ Add follow-up visit"; filter chips: Due this week · Overdue · All upcoming · History; "Total: ₹12,500";
   cards: patient, type, due date, amount, badge, buttons Remind (green) and Mark paid. Empty: "Nothing here".
10. ADD FOLLOW-UP (dialog): For, Type, Due date, Amount (₹, optional), Notes; Save.
11. TASKS: "+ Task"; filter chips: Today · Overdue · Upcoming · Done; each task: checkbox, title (long titles wrap), due date/time,
    linked patient, assigned person. Empty: "No tasks here". ADD TASK dialog: Task, Remind me at, For (optional patient); Save.
12. IMPORT EXCEL: "Upload an Excel (.xlsx) or CSV file. The first row should have column names like Name, Phone, Email." + "Choose file";
    then a column-matching step: each column of the file → a field (Name, Phone, Email, Tags, an industry field) or "Skip this column",
    with 3 preview rows; "Import 240 patients"; result "236 imported · 3 skipped as duplicates · 1 skipped without a name";
    up to 5,000 rows; button "Continue setup" when opened from the setup wizard.
13. PHONE "MORE" MENU: bottom sheet showing business name · user name, then Pipeline, Tasks, Settings, and "Log out" in red.
```

---

## Prompt 4 — settings and special states

```text
Same style. Phone (375px) and desktop (1280px).

SETTINGS with 6 tabs (wrap to 2 lines on phone): Business · Team & devices · Stages · WhatsApp messages · Plan & billing · Data.
A note "Only the account owner can change these settings." appears for the second person (Member).
1. Business: Business profile (Name, Phone, City, Industry shown read-only), Save, "Saved ✓";
   Custom fields (new screen — the backend already supports it): add up to 20 extra fields for contacts
   (label + type: Text / Number / Date / Dropdown with options / Phone), change their order, delete;
   Storage card: "0.4 GB of 20 GB used (20 GB, shared by both people)" with a bar.
2. Team & devices: People (max 2): name, email, role badge (Owner/Member), "Remove" for the member;
   "Invite your partner": email + Invite, then a copyable invite link + "Share on WhatsApp"; pending invites with "Cancel";
   Logged-in devices: "Each person can be logged in on 1 phone and 1 browser." — list: device type 📱/💻, person, last seen, "Log out".
3. Stages: list of pipeline stages in order, each with an editable name and a delete button; "New stage" field + Add;
   the final stages show a Won or Lost marker (e.g. Completed = Won, Not coming = Lost).
4. WhatsApp messages: list of ready messages (name + text with placeholders like {name}, {date}, {time}, {amount}); edit and save.
5. Plan & billing: "₹599 / month + GST"; Status badge (Free trial until 7 Nov 2026 / Active until … / Trial ended / Payment pending /
   Cancelled); "2 people · 20 GB storage shared · all features"; box "To activate, pay ₹599/month + GST by UPI or bank transfer and send us the
   payment details on WhatsApp. We activate your account the same day." + green "Message us on WhatsApp".
6. Data: "Export all data" (Download Excel); "Activity log" (Show recent changes → list: time, person, action);
   "Privacy and terms" links (Terms, Privacy Policy, Data Processing Agreement, Sub-processors, Contact the Grievance Officer);
   "Delete account" danger card: explanation, "Type <business name> to confirm" field, red "Delete account" (disabled until typed);
   after request: red card "Account deletion requested — Everything will be erased on 31 Oct 2026" + "Cancel deletion".

SPECIAL STATES (show each once):
- Top notice bars: trial ending in N days (+ WhatsApp activate link); plan ended — "You can still view your data" (view-only:
  add/edit buttons disabled); payment pending; "This account will be deleted on 31 Oct 2026. You can cancel this in Settings → Data.";
  "Your plan has ended. All data will be deleted on … unless you renew."
- "You were logged out because this account was used on another device." (login screen message)
- Loading (spinner/skeleton), empty states, error message style, success toast "Saved ✓", confirm dialog for Delete.

LEGAL PAGES: one simple reading layout for Terms of Service / Privacy Policy / Data Processing Agreement / Sub-processors:
logo header, a yellow "Draft" notice bar, long text with headings, numbered clauses and tables (tables scroll sideways on phone),
"Version 2026-10-24-draft", same footer.
```

---

## Checklist: every feature must appear in a design

- [ ] Landing, Log in (+Google), Sign up (2 steps + agree checkbox), Forgot password, Set new password, Join team (invite)
- [ ] Setup wizard: 5 steps
- [ ] Dashboard: 4 industry cards, today's appointments, tasks, follow-ups due
- [ ] Contacts list (search, stage filter, pagination), Add/Edit contact (industry and custom fields, tags), duplicate warning + merge
- [ ] Contact page: call, WhatsApp, stage, tags, delete, appointments, follow-ups, tasks, files, notes, history
- [ ] Call outcome dialog; WhatsApp message dialog
- [ ] Pipeline (drag between stages)
- [ ] Appointments: day/week, book, Remind/Done/No-show/Cancel
- [ ] Follow-ups: 4 filters, total ₹, Remind / Mark paid, add
- [ ] Tasks: 4 filters, add, complete
- [ ] Import Excel with column matching
- [ ] Phone bottom bar + More menu; desktop sidebar
- [ ] Settings: Business (+custom fields, storage), Team & devices, Stages, WhatsApp messages, Plan & billing, Data (export, activity log, privacy links, delete account)
- [ ] Notice bars (trial, view-only, payment pending, deletion), logged-out-elsewhere message, loading/empty/error/success states
- [ ] Legal pages layout + footer with Grievance Officer link
- [ ] Words fit for all 3 industries (Patients/Members/Students, Appointments/Sessions, Follow-ups/Renewals/Fees)

**Not in phase 1, so don't design these now:** Razorpay "Subscribe" and "Payment link" buttons (phase 2), Android-only GPS check-in and push notifications (phase 3),
WhatsApp Business API inbox, IndiaMART/JustDial leads, automation rules, Indian-language screens (later).
