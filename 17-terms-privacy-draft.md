# 17 — Terms of Service, Privacy Policy and Data Processing Agreement: DRAFT

> **DRAFT for lawyer review. This is not legal advice and must not be published as written.**
> Written 24 Oct 2026 by the product team so the lawyer has a complete starting point.
> Everything in `[square brackets]` is a placeholder or a decision for VBW or the lawyer.
> The DPDP facts in Part 1 were checked against the official Rules text (sources at the end). The lawyer should still verify them.

**Contents**
1. [Notes for the lawyer](#part-1--notes-for-the-lawyer): facts, roles, open questions, product gaps
2. [Terms of Service](#part-2--terms-of-service-draft)
3. [Privacy Policy](#part-3--privacy-policy-draft)
4. [Data Processing Agreement](#part-4--data-processing-agreement-draft) (VBW as Data Processor for each customer business)
5. [Sub-processor list](#part-5--sub-processors)
6. [Sources](#sources)

---

## Part 1 — Notes for the lawyer

### 1.1 What VBW CRM is

- A web app (Android app later) where **small Indian businesses** keep a list of their own customers:
  hospitals (patients), gyms (members) and coaching centres (students and parents).
- Price: ₹599/month + GST per account; 2 users per account (Owner + Member).
  Free trial **100 days** for every new account; pilot customers then pay ₹599 "locked for life" (see question Q6). Storage 20 GB per account, shared.
- Phase 1 payments: customer pays by UPI or bank transfer and VBW activates the account by hand.
  Phase 2: Razorpay subscriptions and payment links.
- When the trial or paid period ends, the account becomes **view-only**. Today, data is **never deleted** automatically (see gap G3).

### 1.2 Two roles under the DPDP Act

| Personal data | Whose? | VBW's role (our reading, please confirm) |
|---|---|---|
| Account users: owner and staff name, email, login, devices | The business's users | **Data Fiduciary** (VBW decides why and how) |
| The business's own customers: patients, members, students, parents | The customer business | **Data Processor** (VBW processes on the business's behalf). The business is the Data Fiduciary. |

Section 8(2) of the Act lets a Data Fiduciary engage a Data Processor **only under a valid contract**. Rule 6(1)(f) says that contract must provide for reasonable security safeguards. So Part 4 is a processing agreement that each business accepts at sign-up.

### 1.3 Exactly what data the app stores (from the code, 24 Oct 2026)

| Category | Fields |
|---|---|
| Business | Name, phone, city, logo (optional), industry, plan status, trial / paid-until dates, storage used |
| Users | Name, email, role, Firebase login ID; devices (type phone/browser, label, push token, last seen) |
| Contacts (the business's customers) | Name, phone, email, tags, notes, source, pipeline stage. **Industry fields:** hospital: age, gender, doctor, visit reason; gym: plan, trainer, goal, start date; coaching: course, batch, **parent's phone**, total fee |
| Activity | Appointments (type, time, status, notes), renewals / fees (type, due date, amount, status), tasks, call outcomes, WhatsApp messages written from the app (text kept in the timeline) |
| Files | PDF, images, Excel/CSV uploaded by the business (up to 25 MB each) |
| Android app (phase 3) | Optional GPS check-in for field visits, saved as a map link on the contact |
| Logs | Audit log of actions (who did what, when); rate-limit counters by IP address (kept about 1 hour in Redis); server logs |
| Payments (phase 2) | Razorpay IDs and statuses only. **VBW never sees card or UPI details.** |

**Health data:** hospitals may type a "visit reason" and doctor name. Product rule: contact and appointment details only, **no medical records** (also stated in the Terms). Please advise whether "visit reason" should be removed, or limited to a fixed list such as consultation or follow-up (Q3).

**Children:** coaching-centre students and some patients may be **under 18**. The Act requires verifiable parental consent for children's data (s.9). The PIB summary of the Rules says this does not apply where processing relates to essential services such as **healthcare and education**. Please confirm how this applies to a CRM used by hospitals and coaching centres (Q4).

### 1.4 Where data is stored

| What | Where |
|---|---|
| Main database, app servers | Hetzner Cloud, **Singapore** |
| Standby database copy (live replica) | Contabo, **Navi Mumbai, India** |
| Files and encrypted database backups | Cloudflare R2 (choose the Asia-Pacific location when creating the bucket); backups kept about 30 days |
| Login (passwords, Google sign-in) | Google Firebase Authentication. **Storage region not confirmed**: a secondary source says the US by default (see Part 5). |
| Transactional email (password reset, invites, reminders) | Brevo, **EU** (France / Germany / Belgium, per Brevo's help centre) |
| Traffic (TLS, firewall, CDN) | Cloudflare global network |

**Cross-border transfer:** Section 16 of the Act and Rule 15 allow transfer outside India, **subject to any restrictions the Central Government notifies**. Rule 15 also covers requirements on making data available to a foreign State. Commentary we found (not official) reports no country restricted yet. Section 16 and Rule 15 come into force with the 18-month phase (see 1.5).

### 1.5 DPDP timeline (official text)

The DPDP Rules, 2025 were published as **G.S.R. 846(E), dated 13 November 2025** (PIB's summary says "notified on 14 November 2025"). Rule 1:
- Rules 1, 2 and 17–21: in force on publication.
- Rule 4 (consent managers): **one year** after publication (about **13 Nov 2026**).
- Rules 3, 5–16, 22, 23 (notice, security, breach, retention, rights, cross-border and others): **eighteen months** after publication (about **13 May 2027**; law-firm summaries say 12 or 13 May 2027, so please confirm).

Phase 1 launch (Nov 2026) is **before** most duties start. We still want to comply from day one.

Key duties from the official text that shape the drafts:

| Rule | Duty |
|---|---|
| 3 | A notice that stands on its own, in clear and plain language, with an **itemised** list of personal data and the specific purposes. It must also say how to withdraw consent (as easily as it was given), how to exercise rights, and how to complain to the Board. |
| 6 | Reasonable security safeguards, at minimum: encryption/masking, access control, **logs and monitoring**, backups for continuity, **retention of logs and personal data for one year** for investigating unauthorised access, **security clauses in processor contracts**, and organisational measures |
| 7 | Breach: tell each affected person **without delay** (what happened, likely consequences, mitigation, what they can do, contact). Tell the **Board without delay**, with a detailed report **within 72 hours** of becoming aware. |
| 8(3) | Keep personal data, traffic data and processing logs for **at least one year** from processing, for the Seventh Schedule purposes, then erase them unless another law requires longer. Illustration (Case 2) applies this to a cloud provider acting as Data Processor. |
| 9 | Publish on the website/app the business contact of a person who can answer privacy questions |
| 14 | Publish the grievance timeline; respond within **a reasonable period not exceeding 90 days** |

**Penalties** (Act Schedule, per PIB): up to **₹250 crore** for failing to keep reasonable security safeguards; up to **₹200 crore** each for not notifying a breach and for children's-data violations; up to **₹50 crore** for other violations.

### 1.6 Questions for the lawyer

| # | Question |
|---|---|
| Q1 | Do you agree with the Fiduciary / Processor split in 1.2? Is accepting Part 4 by click-through at sign-up a "valid contract" under s.8(2)? |
| Q2 | Our customers (hospitals, gyms, coaching centres) are Data Fiduciaries for their own customers. What must **they** do (notice, consent) before entering their customers' data, and how much should our Terms put on them? |
| Q3 | Hospital fields "visit reason" and "doctor": acceptable as plain contact/appointment data, or treat them as health data and remove or limit them? |
| Q4 | Children (students, child patients): does the healthcare/education exemption cover our customers' use? What should the Terms require of coaching centres and hospitals? |
| Q5 | Storage in Singapore, Firebase possibly in the US, Brevo in the EU: anything to state or do beyond disclosure, given s.16 and Rule 15? |
| Q6 | "₹599 locked for life" for pilots: how to word it safely? Our proposal is "while the account stays continuously paid, excluding taxes and new paid add-ons". |
| Q7 | Retention: how long may we keep (a) data of businesses that stop paying, (b) contacts a business deleted? We propose 90 days and 30 days, see gaps G2/G3. How do these fit the one-year log retention in Rules 6 and 8(3)? |
| Q8 | Liability cap and indemnity wording suitable for small Indian businesses |
| Q9 | Governing law and courts: proposed **[Chennai]** courts. Is an arbitration clause worth it at this price? |
| Q10 | GST invoices, refunds (proposal: no refund for partly used months) and the activation-by-hand process in phase 1 |
| Q11 | Do we need any registration or certificate (for example under the IT Act, 2000 or the IT (Reasonable Security Practices) Rules, 2011) while the DPDP duties are not yet in force? |
| Q12 | WhatsApp: users send messages from **their own** WhatsApp; the app only pre-fills text. Is our clause putting responsibility on the user for WhatsApp's terms and for their recipients' consent enough? |
| Q13 | Offers ([26](26-industries-retail-real-estate.md)): promotional WhatsApp messages go only to customers marked "agrees to get offers" (ticked by staff, or by the customer on the booking page), each message ends "Reply STOP if you do not want offers from us", and every change is kept in the customer's history. Is this enough consent and withdrawal under the DPDP Act and TRAI / WhatsApp rules? Should the booking-page wording change? |
| Q14 | New data: retail birthdays, anniversaries and purchase history; real estate budgets, loan needs and property owners' details. Is the updated privacy-policy field list enough, and does anything need extra care? |

### 1.7 Product gaps to fix (engineering list, not legal text)

| # | Gap | Fix | Why | Status (24 Oct 2026) |
|---|---|---|---|---|
| G1 | No in-app way for an owner to **delete the whole account** | Settings → Data → **Delete account** (owner only, type the business name). Erased 7 days later unless cancelled; database rows, files and logins all go. | Erasure right; Terms promise it | ✅ Built |
| G2 | Deleted contacts are only hidden (`deleted_at`), **never purged** | Nightly job erases contacts deleted more than 30 days ago, with their files | Storage limitation; matches the Privacy Policy | ✅ Built |
| G3 | Data of businesses that stop paying is **kept forever** | After 90 days unpaid: warning emails 30 days and **48 hours** before, then erase. Never erased without both warnings, nor less than 48 h after the final one. | Storage limitation. The 48-hour notice mirrors Rule 8(2) (which formally covers only the Third Schedule classes). | ✅ Built, **switched off** (`LAPSED_ACCOUNT_ERASURE=false`) until Q7 is answered |
| G4 | No server access log kept for 1 year | API writes one line per request (time, IP, method, path without query, status, user, business, device). Nightly: compressed, copied to R2, kept 1 year. | Rules 6(1)(c), 6(1)(e), 8(3) (from May 2027) | ✅ Built; the API refuses to start in production without it |
| G5 | No written breach procedure | [10 — Operations](10-operations.md): who decides, the 72-hour clock, message template | Rule 7 | ✅ Written; names of the two deciders still to fill |
| G6 | No Privacy / Terms pages or consent checkbox | Pages `/terms`, `/privacy`, `/dpa`, `/subprocessors` (from `apps/web/content/legal/*.md`); required "I agree" box at sign-up and invite; version and time stored per person | Rule 3; contract under s.8(2) | ✅ Built, showing the **draft** text with a "Draft" notice |
| G7 | No published privacy contact | Footer on public pages + Privacy Policy + Settings → Data: Grievance Officer (`GRIEVANCE_NAME/EMAIL/PHONE`) | Rules 9 and 14 | ✅ Built; name, email and phone still to set |
| G8 | Firebase Authentication storage region unknown | Check in the Firebase console / Google's data-location docs and update Part 5 | Accurate disclosure | ⏳ At Firebase setup |

**When the lawyer returns the final texts:** replace the four files in `vbw-crm/apps/web/content/legal/`, change `LEGAL_VERSION` in `packages/core/src/constants.ts` (drop `-draft`, which also removes the Draft notice), and set the numbers in `RETENTION` if any period changed. People who signed up earlier keep their recorded version. Ask the lawyer whether they must agree again.

---

## Part 2 — Terms of Service (draft)

**VBW CRM — Terms of Service**
Version [1.0] · Effective [date]

These Terms are an agreement between **[VBW legal entity name]**, [type of entity], [registered address] ("**VBW**", "we", "us"),
and the business that creates a VBW CRM account ("**Customer**", "you"). The person who signs up confirms they are allowed to accept these Terms for the business.

### 1. The service
1.1 VBW CRM is online software for keeping track of a business's customers, appointments, renewals, fees, tasks and files, used through a web browser and (when available) an Android app ("**Service**").
1.2 We may improve or change features. We will not remove a core feature you pay for without at least [30] days' notice.

### 2. Accounts, users and devices
2.1 Each account is for one business and up to **2 users**: one Owner and one Member.
2.2 Each user may be logged in on **one phone and one browser** at a time. A new login on the same kind of device logs out the old one.
2.3 You are responsible for your users, for keeping logins secret, and for everything done through your account. Tell us at once at [support email] if you think a login has been misused.
2.4 The Owner can remove the Member and log out devices at any time.

### 3. Free trial and pilot offer
3.1 New accounts get a free trial of **100 days**, starting when the account is created. It ends exactly 100 days later; after that the account can be viewed and exported but not changed until a plan is paid.
3.2 **Pilot price:** businesses that join under a written pilot offer pay ₹599 per month (plus GST) **for as long as the account stays continuously paid**. If the account lapses for more than [30] days, standard prices apply. New paid add-ons (for example WhatsApp Business API or SMS credits) are charged separately. *[Lawyer: see Q6.]*

### 4. Price and payment
4.1 The price is **₹599 per account per month plus GST**, unless a written offer says otherwise.
4.2 **Phase 1:** you pay by UPI or bank transfer and send us the payment reference. We activate the paid period after we confirm the payment. **Later:** payment may be taken online through Razorpay as a monthly subscription.
4.3 We will issue a GST invoice for each payment. *[Lawyer: GSTIN, invoice timing, Q10.]*
4.4 Fees are for a month at a time. **We do not refund part-used months**, except where the law requires it or we end the Service under clause 10.3.
4.5 We may change prices for new periods with at least [30] days' notice by email. Pilot prices under clause 3.2 do not change.

### 5. When the plan ends
5.1 If the trial or paid period ends, the account becomes **view-only**: you can see and export your data, but not add or change it.
5.2 If the account stays unpaid for [90] days after that, we will email the Owner at least **[30] days** and again **48 hours** before we permanently delete the account's data. You can export everything to Excel at any time before then. *[Product: gap G3.]*

### 6. Your data
6.1 **You own your data.** "Customer Data" means the information you or your users put into the Service, including details of your own customers.
6.2 You give us permission to store and process Customer Data **only to provide the Service** to you, as set out in the Data Processing Agreement (Part 4), which forms part of these Terms.
6.3 **You are responsible for having the right to collect and use your customers' data**, including any notice to them and their consent where the law requires it, and the consent of a parent or guardian for children where required. *[Lawyer: Q2, Q4.]*
6.4 You can export all Customer Data to Excel at any time from Settings → Data.
6.5 We will not sell Customer Data, use it for advertising, or contact your customers.

### 7. Acceptable use
You must not use the Service to:
(a) store **medical records**, prescriptions, test results or diagnoses (hospitals may store contact and appointment details only);
(b) store payment card numbers, bank passwords, Aadhaar numbers or other government ID numbers;
(c) send spam, or message people who have asked you not to contact them;
(d) break the law, including the Digital Personal Data Protection Act, 2023, the Information Technology Act, 2000, and telecom rules on unsolicited messages;
(e) upload viruses or harmful files, or try to access other businesses' data or our systems;
(f) resell or share the Service with another business.

### 8. WhatsApp and calls
8.1 The Service can prepare a WhatsApp message and open **your own** WhatsApp app. You decide whether to send it. **The Service does not send WhatsApp messages for you.**
8.2 You are responsible for following WhatsApp's terms and for having your recipients' permission to message them.

### 9. Availability, support and backups
9.1 We aim to keep the Service available at all times, but we do not promise it will never be interrupted. We may pause it for maintenance, preferably between [11 pm and 6 am IST].
9.2 We back up the database every day. *[Lawyer: whether to state recovery targets. Internal target: lose at most about 5 minutes of data, back within a few hours.]*
9.3 Support: WhatsApp [number] and email [support email], [Mon–Sat, 10 am–7 pm IST].

### 10. Ending the agreement
10.1 You can stop using the Service at any time. To delete your account and data, use Settings → Delete account *[gap G1]* or email [support email]. We will delete Customer Data within [30] days, except copies we must keep by law (see the Privacy Policy, retention).
10.2 We may suspend an account that breaks clause 7, after notice where reasonable.
10.3 We may end the Service for everyone with at least [90] days' notice, and refund any unused prepaid fees.

### 11. Liability
11.1 The Service is provided "as is". It is a tool to help you follow up. **You remain responsible for your business decisions and for your customers.**
11.2 Neither party is liable for indirect or consequential loss, or lost profits.
11.3 Our total liability for any claim is limited to **the fees you paid us in the [12] months before the claim**. *[Lawyer: Q8.]*
11.4 Nothing in these Terms limits liability that cannot be limited under Indian law.

### 12. Changes to these Terms
We may update these Terms. For important changes, we will email the Owner at least [30] days in advance. Continuing to use the Service after that date means you accept the new Terms.

### 13. Law and disputes
These Terms are governed by the laws of India. Courts at **[Chennai]** have exclusive jurisdiction. *[Lawyer: Q9.]*

### 14. Contact
[VBW legal entity name], [address] · [support email] · WhatsApp [number]

---

## Part 3 — Privacy Policy (draft)

**VBW CRM — Privacy Policy**
Version [1.0] · Effective [date]

This policy explains how **[VBW legal entity name]** ("VBW", "we") handles personal data in VBW CRM.
It is written to meet the notice requirements of the Digital Personal Data Protection Act, 2023 and the DPDP Rules, 2025.

### 1. Two kinds of data, two roles

| Whose data | Who decides how it is used | Our role |
|---|---|---|
| **You, as a user** of VBW CRM (business owner or staff) | VBW | We are the **Data Fiduciary**. This policy covers it fully. |
| **Your customers** (patients, members, students, parents) whose details your business enters | Your business | Your business is the Data Fiduciary and we are its **Data Processor**. We use this data only as the business instructs (see the Data Processing Agreement). **If you are a customer of a business that uses VBW CRM, please contact that business first** about your data. |

### 2. Data we collect about users (itemised)

| Data | Why we use it |
|---|---|
| Name, email address, role (owner/member) | Create and run your account; send login, invite and service emails |
| Password (held by Google Firebase, not by us) or Google sign-in | Let you log in securely |
| Business name, phone, city, logo | Show your business in the app and on invoices |
| Device type, device label, push-notification token, last-seen time | Enforce "1 phone + 1 browser per user"; send reminders to your phone (Android app) |
| IP address and request logs | Security: rate limits, detecting misuse, investigating incidents |
| Payment references (UPI/bank reference; later Razorpay IDs) | Activate your plan and keep accounts. **We never see card or UPI PINs.** |
| Messages you send us on WhatsApp or email | Support |

We do **not** use your data for advertising, and we do **not** sell it.

### 3. Data your business stores about its customers
Whatever your business enters: names, phone numbers, emails, appointments, renewals, fees, notes, tasks, files, and industry fields
(hospital: age, gender, doctor, visit reason; gym: plan, trainer, goal, start date; coaching: course, batch, parent's phone, fees).
We process it only to provide the Service to that business.

### 4. Legal basis
- **User data:** your consent, given when you sign up and tick the agreement box. You can withdraw consent by deleting your account (see section 9). Some processing is also needed to provide the Service you asked for and to meet legal duties (for example tax records). *[Lawyer: confirm the legitimate uses under s.7 that apply.]*
- **Customers' data:** the business that entered it is responsible for having a lawful basis.

### 5. Who we share data with
Only with the service providers listed below, which help us run VBW CRM, and only as needed. A full, current list is at [link to Part 5 page].

| Provider | What for | Where |
|---|---|---|
| Hetzner Online GmbH | Main servers and database | Singapore |
| Contabo GmbH | Standby copy of the database | Navi Mumbai, India |
| Cloudflare, Inc. | Website security and speed; file and backup storage (R2) | Global network; storage in Asia-Pacific |
| Google LLC (Firebase) | Login; push notifications (Android app) | [To confirm, possibly USA] |
| Brevo (Sendinblue SAS) | Sending emails | European Union |
| Razorpay Software Pvt Ltd *(from phase 2)* | Online payments | India |

We may also share data when the law requires it, for example on a valid order from a court or government authority.

### 6. Where data is stored
Mainly in **Singapore**, with a live copy in **India (Navi Mumbai)**. Some providers above process data in other countries.
We will follow any restrictions the Government of India notifies under section 16 of the DPDP Act.

### 7. How we protect data
- Each business can see only its own data. This is enforced inside the database (row-level security), not just in the app.
- Encrypted connections (HTTPS) everywhere; files kept in private storage and opened only through short-lived links.
- Passwords are handled by Firebase; we never store them.
- Daily encrypted backups, and a standby copy in another country.
- Access to servers only by named staff with security keys; actions in the app are recorded in an audit log.

### 8. How long we keep data *[Lawyer: Q7]*

| Data | Kept for |
|---|---|
| Account and customer data | While the account is active |
| After the plan ends | View-only for [90] days, then deleted after email warnings (30 days and 48 hours before) |
| Contacts a business deletes | Hidden at once, permanently erased after [30] days *[gap G2]* |
| Backups | About 30 days, then overwritten |
| Security and processing logs | [1 year], as required by the DPDP Rules, then erased |
| Invoices and payment records | As long as tax law requires *[Lawyer: years]* |

### 9. Your rights
Under the DPDP Act you can ask us to:
- tell you what personal data we hold about you and how we use it;
- **correct, complete or update** it;
- **erase** it (we may keep what the law requires);
- **nominate** someone to exercise your rights if you die or cannot act yourself;
- **withdraw consent**, as easily as you gave it, by deleting your account or writing to us.

Write to our Grievance Officer (below). We will reply within **[30] days**, and never later than **90 days**.
If you are not satisfied, you can complain to the **Data Protection Board of India**.

If your data was entered by a business that uses VBW CRM (for example your hospital or gym), contact that business. We will help them respond.

### 10. Personal data breach
If a breach affects your personal data, we will tell you without delay: what happened, the likely effect on you, what we are doing, and what you can do.
We will also report it to the Data Protection Board as the DPDP Rules require. For customers' data, we will tell the affected business without delay so it can meet its own duties.

### 11. Children
VBW CRM user accounts are for adults running a business. Businesses may store details of children (for example students or young patients);
the business is responsible for any parental consent required. *[Lawyer: Q4.]*

### 12. Changes
We will post updates here and email account owners about important changes at least [30] days before they apply.

### 13. Contact and Grievance Officer
**[Name]**, Grievance Officer, [VBW legal entity name], [address]
Email: [privacy@domain] · Phone/WhatsApp: [number] · [Mon–Sat, 10 am–7 pm IST]

---

## Part 4 — Data Processing Agreement (draft)

Between the **Customer** (the business; Data Fiduciary) and **VBW** (Data Processor). It forms part of the Terms of Service and is accepted at sign-up.

1. **Scope.** VBW processes Customer Data (the personal data of the Customer's own customers and staff) only to provide VBW CRM, and only on the Customer's documented instructions. The Terms and the use of the app's features count as those instructions.
2. **Purpose limits.** VBW will not use Customer Data for its own purposes, advertising or sale, and will not contact the Customer's customers.
3. **Staff.** Only VBW staff who need access to run or support the Service may access Customer Data, and they are bound by confidentiality. Support access happens only when the Customer asks for help, or to investigate a security problem.
4. **Security (Rule 6).** VBW keeps reasonable security safeguards, at least: encryption in transit and for backups; database-level separation of each business's data; access control with individual keys; logging and monitoring of access; daily backups and a standby copy; and organisational measures such as least access and staff training. Details are in [07 — Security and reliability] (to be shared as an annex).
5. **Sub-processors.** The Customer authorises the sub-processors in Part 5. VBW will give [30] days' email notice before adding or replacing one, and the Customer may end the agreement if it objects. VBW requires each sub-processor to protect data at least as well as this agreement.
6. **Transfers.** Customer Data is stored mainly in Singapore and India, and some sub-processors operate elsewhere (Part 5). VBW will follow any restriction notified under section 16 of the DPDP Act.
7. **Breach.** VBW will tell the Customer **without undue delay, and within [24] hours** of becoming aware of a breach affecting Customer Data. It will give the facts the Customer needs to inform affected people and to report to the Board within the **72-hour** period in Rule 7(2)(b).
8. **Data principals' requests.** VBW will help the Customer answer access, correction and erasure requests, mainly through in-app editing, deletion and Excel export.
9. **Logs.** VBW keeps processing logs for [one year] (Rule 8(3), Case 2) and makes relevant extracts available to the Customer on request.
10. **Return and deletion.** At any time the Customer can export all Customer Data. When the account is closed or deleted under the Terms, VBW erases Customer Data within [30] days, and from backups when they expire (about 30 days). Logs are kept as clause 9 requires.
11. **Audit.** On reasonable written request, at most once a year, VBW will answer a security questionnaire and share summaries of its security measures.
12. **Liability.** As in the Terms of Service.

---

## Part 5 — Sub-processors

*To publish at `/subprocessors` and keep current. Locations must be verified at setup (gap G8).*

| Sub-processor | Service | Data involved | Location (as researched, Oct 2026) |
|---|---|---|---|
| Hetzner Online GmbH (Germany) | Cloud servers: app, database, cache | All | Singapore data centre |
| Contabo GmbH (Germany) | Standby database server | All (database copy) | Navi Mumbai, India (Contabo's India location, operated by NTT DATA) |
| Cloudflare, Inc. (USA) | DNS, TLS, firewall/CDN (all traffic passes through); R2 storage for files and encrypted backups | All in transit; files; backups | Global network; R2 bucket created with an Asia-Pacific location hint *[confirm at setup]* |
| Google LLC: Firebase Authentication | Login, password reset links, Google sign-in | User name, email, login IDs, IP for login | **Not confirmed.** A secondary source says US by default; check Google's Firebase data-location documentation. |
| Google LLC: Firebase Cloud Messaging *(Android app, phase 3)* | Push notifications | Device token, notification text | [To confirm] |
| Brevo SAS (France) | Transactional email | User emails, names, business names in emails | EU (OVH in France/Germany; Google Cloud in Belgium), per Brevo's help centre |
| Razorpay Software Pvt Ltd *(phase 2)* | Subscriptions and payment links | Business contact for billing; customers' name and phone for payment links | India |

Not sub-processors: **WhatsApp** (users send messages from their own app; VBW only opens a pre-filled link) and **GitHub** (code only, no customer data).

---

## Sources

Official:
- DPDP Rules, 2025 (G.S.R. 846(E), 13 Nov 2025), English text: Rules 1, 3, 6, 7, 8, 9, 14, 15 quoted above. [MeitY PDF](https://www.meity.gov.in/static/uploads/2025/11/53450e6e5dc0bfa85ebd78686cadad39.pdf)
- PIB backgrounder "DPDP Rules, 2025 Notified" (17 Nov 2025): penalties (₹250 / ₹200 / ₹50 crore), 90-day response, children's-data exemption for healthcare and education. [PIB PDF](https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf)
- DPDP Act, 2023 text. [MeitY PDF](https://www.meity.gov.in/static/uploads/2024/06/2bf1f0e9f04e6fb4f8fef35e82c42aa5.pdf). Sections 7, 8, 9 and 16 are cited from it. *Lawyer to verify the exact wording.*

Commentary (not official):
- AZB & Partners, phased rollout table (phase 3 date May 12, 2027). [AZB](https://www.azbpartners.com/bank/indias-digital-personal-data-protection-act-phased-rollout-and-key-compliance-milestones/)
- Cross-border status (no country restricted yet). [Legal500](https://www.legal500.com/intelligence/india/privacy/cross-border-data-transfers-under-indias-dpdp-act-rules-restrictions-and-compliance-roadmap-for-2027), [advaslam.com](https://www.advaslam.com/writing/dpdp-cross-border-data-transfer-section-16/)

Providers:
- Firebase privacy and security (certifications, retention of auth data). [firebase.google.com/support/privacy](https://firebase.google.com/support/privacy). Storage region of Auth is **not stated there**. A secondary claim of US storage: [simpleanalytics.com](https://www.simpleanalytics.com/is-gdpr-compliant/firebase)
- Brevo data storage location. [help.brevo.com](https://help.brevo.com/hc/en-us/articles/360001005510-Data-storage-location)
- Contabo India location (Navi Mumbai). [contabo.com/en-us/locations/india](https://contabo.com/en-us/locations/india), [NTT DATA case study](https://www.nttdata.com/global/en/insights/reports/contabo)
