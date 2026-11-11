# 19 — Claude Design: public pages + login animation (combined Prompt 2)

This **replaces Prompt 2 in [18](18-claude-design-prompts.md)**. It combines:
1. every public-page screen from Prompt 2 (landing, log in, sign up, forgot / new password, join team, setup wizard),
2. the "VBW Pulse" animation theme from the current prototype (teal band, pulse line, faint appointment-book text, light + dark),
3. a **liquid glass** material for the cards, fields and buttons (reusable style, flat fallback for cheap phones),
4. the split-screen idea from the second reference picture (industry showcase on the left, glass login card on the right,
   floating words flowing into the card, theme toggle on the card).

**Attach both pictures** when you send it: picture 1 = current prototype (the theme), picture 2 = layout reference.

Things deliberately NOT copied from picture 2:
- The brand "GlobalConnect" and its logo; the stock photos of real people (we use illustrations, or made-up pictures we own).
- Its industries "[Real Estate], Finance, Manufacturing, E-commerce…": VBW CRM supports **Clinic, Gym and Education Centre** in phase 1.
  Real Estate is a later phase, so it is not shown.
- Its "[...]" placeholder brackets.

If Claude Design cuts the answer short or the result gets messy, send the prompt in two parts at the line `----- SPLIT HERE -----`.

---

## The prompt

```text
Design the PUBLIC side of "VBW CRM" (a simple CRM for small Indian clinics, gyms and coaching centres) as ONE connected set.
Attached: picture 1 = my current prototype (this is the THEME to keep), picture 2 = a layout reference (take only the layout idea;
do NOT copy its brand name, logo or stock photos of people; do NOT use its industries — mine are only Clinic, Gym, Education Centre).
Show every screen at phone (375px) and desktop (1280px), in LIGHT and DARK mode, with still frames for each animation state.

THEME (from picture 1): teal band across the top with a thin white "VBW Pulse" heartbeat line running through it, white card overlapping
the band, rounded pill buttons, friendly sans-serif, faint appointment-book text ("9:30 Priya", "10:15 Arun") on the empty sides.
Colours: teal #0f766e, WhatsApp green #25D366 only on WhatsApp buttons, red for errors only. Dark mode = deep teal-black background,
card slightly lighter, text and teal still clear. Footer on every public page: Terms of Service · Privacy Policy ·
Data Processing Agreement · Sub-processors · "Privacy questions or complaints: Grievance Officer" · © VBW.

LAYOUT (from picture 2, adapted):
- DESKTOP login / sign-up / join-team: split screen.
  LEFT half = industry showcase: three big illustrated panels or a slow slideshow — Clinic (stethoscope, appointment book),
  Gym (dumbbell, membership card), Education Centre (books, classroom board) — each with its one-line promise:
  "Never miss a patient follow-up" · "Stop losing members at renewal time" · "Every enquiry followed up, every fee collected on time".
  Flat or soft illustrations only, no photos of real people.
  Floating words in a gentle flowing ribbon drift from the left and ride the pulse line into the card:
  Patients · Members · Students · Appointments · Sessions · Follow-ups · Renewals · Fees · WhatsApp.
  RIGHT half = a glass-style card on the teal stage with the form. Small Light / Dark / Auto theme toggle in the top-right of the card.
- PHONE: no left half. The teal band on top holds the logo, the pulse line and one rotating industry line;
  the card sits below, overlapping the band. The form is always fully usable.

ANIMATION CONCEPT "THE VBW PULSE":
- Page open: card slides up; the pulse draws left to right and beats slowly; floating words start drifting.
- Four small dots on the pulse = the form fields; the active dot lights up when its field is focused and the beat speeds up a little.
- Typing the business name (sign-up): the pulse quietly changes shape — heartbeat for clinic/hospital/dental, rep wave for gym/fitness,
  rising bars for classes/academy/institute; default heartbeat. The matching industry panel on the left highlights.
- "I agree" checkbox: quick teal tick; the red warning fades out.
- Error (wrong password, empty box): pulse flatlines for a second, card gives a gentle shake, red text with an icon.
- Loading: button text turns into a tiny beating pulse.
- Success: pulse turns into a tick, card lifts away, greeting fades in on the band.
- Reduce motion: provide a still version with no movement. Must be light enough for cheap Android phones (CSS/SVG only, no video, no 3D).

MATERIAL "LIQUID GLASS" (applies to all public pages; the theme, screens, fields and wording above do NOT change):
- The form card, the field boxes, the main button, the Google button, the Light/Dark/Auto toggle and the setup choice cards are clear
  liquid glass: translucent, softly blurred, slightly refracting what is behind them (the pulse line, floating words, appointment-book
  text), with a thin bright edge highlight on the top-left, a faint inner shadow at the bottom-right, rounded corners (24px card, pill buttons)
  and a subtle teal tint. Dark mode = dark glass with a cooler edge light.
- Liquid motion: when a field is focused a soft glass "droplet" highlight glides to it; the card edge shimmers slowly; on error the card
  wobbles like liquid and settles; on success the card melts and flows into the greeting. The pulse line bends slightly as it passes behind the glass.
- Legibility first: all text and field borders keep strong contrast; labels and typed text never sit on a busy background (the glass
  is more frosted behind text); error messages are solid red text on a frosted chip.
- Cheap phones and accessibility: glass only on the card and button surfaces (never full-page blur); provide a flat frosted fallback
  (no blur, no refraction) for low-end phones and "reduce transparency"; no motion for "reduce motion".
- Deliver it as ONE reusable "liquid-glass" style (blur, tint, edge highlight, shadow, radius) used identically on every public page,
  with small / medium / large variants.

SCREENS (keep every field and button exactly):
1. LANDING PAGE
   - Top bar: "VBW CRM" logo text, "Log in" button, "Start free trial" button, theme toggle.
   - Hero: "Follow up every patient, member and student — on time." Sub-text: appointments, renewals, fee reminders and WhatsApp in
     one place, on web and Android. Buttons "Start 100-day free trial" and green "Book a demo on WhatsApp".
     Small line: "Set up in 5 minutes · ₹599/month for 2 people". The pulse line and floating words run through the hero.
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
   - Footer as above.
2. LOG IN: email, password (show/hide eye), "Log in" button, "Log in with Google" button, "Forgot password?" link,
   "New to VBW CRM? Start free trial" link. Error area (e.g. "Wrong email or password"). Message variant:
   "You were logged out because this account was used on another device." Success greeting on the band:
   "Good morning, Priya. You have 6 appointments today."
3. SIGN UP — step 1 "Start your free trial — 100 days free. No card needed": Your name, Business name (placeholder "e.g. Sri Sai Clinic"),
   Email, Password (8 or more characters, show/hide), REQUIRED checkbox "I agree to the Terms of Service and Privacy Policy, and to the
   Data Processing Agreement for my business" (the three names are links) with error "Please tick the box to continue",
   "Start free trial" button, "or" divider, "Sign up with Google", "Already have an account? Log in".
   Step 2 "Finish creating your account — Logged in as name@email.com": Your name, Business name, the same checkbox, "Create account".
   Success greeting: "Welcome, Priya. Let's set up Sunrise Family Clinic."
4. FORGOT PASSWORD: email field, "Send reset link"; success "If this email has an account, we sent a link" — the pulse folds into
   an envelope and flies off. "Back to log in".
5. SET NEW PASSWORD (from the email link): New password (8+ characters), Type it again, strength line that turns grey to teal,
   "Save password"; states: "Checking link…", "This link is incomplete. Request a new reset link.",
   "The two passwords do not match.", success "Password changed — You can now log in".
6. JOIN YOUR TEAM (invite link): "Join your team on VBW CRM — Use the email address the invite was sent to": Your name, Email,
   Password (8+ characters), the agree checkbox (Terms + Privacy only), "Join", toggle "I already have a password" / "I am new here".
   Two pulse lines meet and join into one on success.
----- SPLIT HERE -----
7. SETUP WIZARD (after sign-up), 5 steps with a progress bar made of the pulse line; on phone show "Step 2 of 5 · Your business":
   Step 1 "What kind of business is it?" — 3 big choice cards: Clinic 🩺, Gym & Fitness 🏋️, Education Centre 🎓 (with their one-line pitch);
   choosing one changes the pulse shape and the words (Patients / Members / Students).
   Step 2 "About your business": Business name, Phone, City; Back / Next.
   Step 3 "Bring in your patients" (word changes per industry): "Import from Excel / CSV" button, "Skip for now".
   Step 4 "Invite your partner — Each account is for 2 people, for example you and your receptionist": email field + "Invite", Back / Skip.
   Step 5 "You are ready! 🎉" + "Go to my dashboard" (pulse turns into a tick, soft confetti).

DELIVER: a still frame for each state (idle, focused field, typing, error, loading, success) for Log in, Sign up and Forgot password,
a short motion note (timing and easing) for each animation, and the light and dark version of every screen.
```

---

## After Claude Design answers

1. Check each screen against the list above (7 screens, light + dark, phone + desktop). Fix with comments.
2. Save screenshots (and exported code if available) in `C:\Users\Surya\Downloads\car web\design\auth\`.
3. Then send **Prompt 3** and **Prompt 4** from [18](18-claude-design-prompts.md), starting with: "Same style as the screens above. Light and dark."
4. Tell Claude Code "build it". It will build these pages into `vbw-crm/apps/web`, keeping login, the "I agree" box,
   the legal links and all security checks working, and test light/dark, phone/desktop and reduced motion.
