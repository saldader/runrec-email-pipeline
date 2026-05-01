# START HERE — RunRec Klaviyo Implementation Package

**Last updated:** 2026-05-01
**Owner:** RunRec marketing operations
**Audience:** The admin who is going to actually click the buttons in Klaviyo

---

## What this package is

This is a complete, plug-and-play guide to building **13 RunRec email and SMS automated flows** inside Klaviyo. "Flows" are the automated email and text sequences that go out to customers automatically when something happens — for example, when someone books a court for the first time.

You do not need to invent anything. Everything has been pre-written:

- Every email's subject line, preview text, and body copy
- Every text message
- Every delay (how long to wait between messages)
- Every filter (who should and should not receive each message)
- Every test step before you turn it on

Your job is to follow the guides in order and click the buttons.

---

## Who this is for

This package is written for someone who has **never used Klaviyo before** and has very little email-marketing background. Every term is defined the first time it appears (see the Glossary at the end of this file). Every step is click-by-click.

If you have used Klaviyo before, you can skim faster — but do not skip the Pre-Flight Setup file. Most of what's in there is missing from the current account.

---

## What you need before you start

| Requirement | Why |
|-------------|-----|
| Klaviyo account access at **Owner** or **Admin** role | Only Owners/Admins can change account settings, set up sending domains, and create custom profile properties |
| Two-factor authentication (2FA) enabled on your Klaviyo login | Required by Klaviyo for admin-level work; protects the customer list |
| Access to the email pipeline dashboard at **https://saldader.github.io/runrec-email-pipeline/** | This is where every email's exact copy lives. Each flow guide tells you which dashboard URL to copy from. |
| Access to the master buildsheet at `~/vault/runrec/marketing/klaviyo-master-buildsheet-v2.md` | Background reference for architecture decisions. You won't read it cover-to-cover, but the per-flow guides reference it. |
| Access to the audit document at `~/vault/runrec/marketing/audit-2026-05-01.md` | Lists known issues and the recommended launch order |
| Approximately **3 to 5 days of focused work** | Spread across two weeks if needed, but do not stretch beyond that — partial states are risky |
| Sal (the operator) reachable for decisions | A handful of decisions can only be made by Sal — flagged in the relevant guide |

---

## Order to follow — DO NOT SKIP STEPS

This is a strict order. Each step depends on the one before it.

### Step 1 — Read and complete `01-pre-flight-setup.md`

This is the heaviest document in the package. It covers everything that has to exist in Klaviyo *before* any flow can be built: domain authentication, custom profile properties, lists, segments, custom events, templates, and account settings.

**Do not start Step 2 until every checkbox in the Pre-Flight Checklist (end of file 01) is green.**

### Step 2 — Read `02-build-template.md`

This is the structure that every per-flow guide follows. It tells you what to expect when you open one of the `flows/XX-<flow-name>.md` files. Read it once, all the way through. Don't skim. Once you know the template, you'll move through each flow much faster.

### Step 3 — Build the 13 flows in this exact order

The order is from the audit's recommended launch sequence. We start with the lowest-risk, highest-engagement flows so the warm subscribers see clean automated emails first. We end with the most complex flow (Corporate, Section 9) once you're comfortable with the pattern.

| Build order | Flow # | Flow name | Why this position |
|-------------|--------|-----------|-------------------|
| 1 | 02 | Booking Confirm + Reminders | Transactional, highest open rate, lowest risk of complaint |
| 2 | 03 | Post-Session + Review | Builds engagement signal on warm list |
| 3 | 13 | Sunset / Hygiene | Clean the list before driving high volume |
| 4 | 12 | Abandoned Booking | Highest revenue-per-recipient flow in the system |
| 5 | 04 | Customer Birthday | High emotion, low volume, great brand moment |
| 6 | 01 | Welcome (existing — extend, don't rebuild) | Now your warm-list signals are strong; safe to expose new signups |
| 7 | 06 | Win-Back (existing — extend) | Reactivates dormant bookers |
| 8 | 05 | Birthday Party Anniversary | Specific audience, low volume |
| 9 | 07 | VIP + Referral | Requires VIP segment to be populated first |
| 10 | 08 | Lead Nurture | Requires Welcome to be live (anti-overlap rule) |
| 11 | 11 | Membership Lifecycle | Requires Stripe subscription events wired |
| 12 | 10 | Membership Conversion | Requires VIP/Power User segment populated |
| 13 | 09 | Corporate & League | Most complex. Multi-touch, B2B, location-manager-bound. |

For each flow:

1. Open the matching guide at `flows/XX-<flow-name>.md`
2. Follow it top to bottom
3. Run the test plan at the end of the guide
4. Update the activation checklist
5. Move to the next flow

### Step 4 — Read and run `04-test-protocol.md`

Before any flow goes live to real customers, this is the universal test pass. It catches the kind of mistakes that turn into "why did the customer get four identical texts" tickets.

### Step 5 — Read and run `05-go-live-checklist.md`

The final cross-flow verification. Sender domain, suppression, send-time windows, frequency caps. Once this is green, you can switch flows from Draft to Live one at a time.

### Reference any time

- `06-compliance.md` — CASL (Canada), CAN-SPAM (US), TCPA (SMS), Klaviyo's terms. Read once early; consult when you have a specific question.
- `07-troubleshooting.md` — When something looks wrong. Symptom-based: "Customer says they got the same email twice." "Open rate dropped from 50% to 12% overnight." "Klaviyo says my flow isn't triggering."

---

## If something goes wrong — decision tree

| Symptom | Where to go first |
|---------|------------------|
| I don't understand what a Klaviyo term means | Glossary at the end of this file |
| The Pre-Flight Setup is asking me to do something I don't know how to do | Escalate to Sal — pre-flight has technical setup that may need an engineer |
| A flow's per-touch copy looks different on the dashboard than what's in the guide | The dashboard at https://saldader.github.io/runrec-email-pipeline/ is canonical for copy. The guide is canonical for build steps. If they conflict, ask Sal. |
| Klaviyo says I can't create a custom event | Custom events come from outside systems (Skedda, Stripe). Escalate to Sal — engineer involvement needed. |
| A test send to my own email looks broken (missing logo, broken link) | `07-troubleshooting.md`, then re-run the test plan in the flow guide |
| I'm about to click a button I'm not sure about | **Stop. Ask Sal.** Email and SMS sends are irreversible. |
| Customer got an email they shouldn't have | `07-troubleshooting.md` first, then escalate to Sal immediately. Pause the flow. |

---

## Time estimates per flow

These are realistic. Add 25% if it's your first time using Klaviyo.

| Flow complexity | Examples | Build time | Test time |
|-----------------|----------|-----------|----------|
| Simple (2-3 touches, 1 trigger, no branches) | 13 Sunset, 12 Abandoned Booking | 30-45 min | 20 min |
| Medium (4-5 touches, 1-2 branches) | 02 Booking, 03 Post-Session, 04 Birthday, 05 Anniversary, 06 Win-Back, 10 Membership Conversion | 60-90 min | 30 min |
| Complex (6-7 touches, multiple branches, location merge tags) | 01 Welcome, 07 VIP, 08 Lead Nurture, 11 Membership Lifecycle | 90-120 min | 45 min |
| Most complex (multi-location, B2B, location-manager-bound) | 09 Corporate & League | 2-3 hours | 60 min |

**Total realistic build time:** 18-26 hours of clicking, plus 8-10 hours of testing. Plan for **3 to 5 working days** of focused work. Do not try to do it in one day — fatigue causes mistakes, and mistakes in email marketing are visible to thousands of people.

---

## Don't-do list

These are the buttons and actions that can cause real damage. **Never click these without explicit approval from Sal.**

| Never do this | Why |
|---------------|-----|
| Click "Send Now" on any campaign | All sends require pre-send checklist. We do not send from this account ad-hoc. |
| Click "Delete Profile" on any customer record | Deletes their history forever. Use Suppress instead if you need to stop sending to them. |
| Click "Bulk Action" without a filter applied | You can accidentally apply an action to every profile in the account. Always confirm the count is what you expect. |
| Edit a flow that is currently Live (active) | Customers who are mid-flow can get broken or duplicate sends. Always clone the flow, edit the copy, then swap. |
| Skip the test protocol for any flow | Every flow gets at least one full live-data test before activation. No exceptions. |
| Move a flow from Draft to Live without the activation checklist complete | The checklist exists because experienced operators have already made the mistakes you would otherwise make. |
| Disable Smart Sending without understanding it | Smart Sending prevents the same person from getting hit twice. Turning it off can cause a spam-complaint storm. |
| Send to "All Profiles" or "Everyone" | Broadcasts must always be filtered to engaged subscribers. Sending to dead addresses tanks deliverability. |
| Change the sender domain or DNS records yourself | Domain authentication is a one-way door. Coordinate with Rob (engineer) and Sal. |
| Add a discount code or pricing | Pricing is operator-only. The Mechanic Doctrine in the buildsheet says reply-to-redeem, not codes. |

---

## Glossary

The minimum vocabulary to navigate Klaviyo. Each term is defined the first time it appears in the per-flow guides as well.

| Term | What it means |
|------|---------------|
| **Profile** | One person in Klaviyo. Identified by email and/or phone number. |
| **List** | A static collection of profiles. Profiles are added or removed manually, by signup form, or by import. Example: "Newsletter," "Tri-City Members." |
| **Segment** | A *dynamic* collection of profiles defined by rules (filters). Klaviyo updates the segment automatically as profiles match or stop matching. Example: "Engaged in last 30 days." |
| **Property** (a.k.a. "profile property" or "custom property") | A piece of information attached to a profile. Built-in: first name, email. Custom: location, last booking date, child name. |
| **Event** (a.k.a. "metric") | Something that happened to a profile. Built-in: opened email, clicked link. Custom: booking_created, corporate_inquiry. |
| **Flow** | An automated sequence of emails and texts that fires when a trigger happens. |
| **Trigger** | The event or action that starts a flow. Examples: profile added to list, custom event "booking_created," date matches profile's birthday. |
| **Filter** (in a flow) | A rule that decides whether a profile continues through the flow at a given point. Example: "Only continue if the profile has NOT made a booking in the last 7 days." |
| **Conditional Split** | A branch inside a flow. Sends profile down Path A or Path B based on a condition. Example: "If they clicked the offer email, send Path A; if not, send Path B." |
| **Template** | A reusable email design. You build the design once, then plug in copy for each send. |
| **Send Test** | Sends a preview of an email to a test address (yourself) without saving the send to history. Always do this before activating a flow. |
| **Smart Sending** | A Klaviyo setting that prevents the same person from receiving the same email or SMS within a short time window (default: 16 hours email, 12 hours SMS). |
| **Quiet Hours** | A Klaviyo setting that prevents sends outside specified hours per recipient's timezone. Standard: 9 PM to 9 AM. |
| **Suppression** | Marking a profile as "do not send to." Different from Unsubscribe. Used for hard bounces, spam complaints, and specific marketing actions. |
| **Sender Profile** | The from-name + from-email combination shown to recipients. Example: "The RunRec <contact@therunrec.com>." |
| **Sender Domain** (a.k.a. "branded sending domain") | The domain Klaviyo uses behind the scenes to send your email. Without your own (e.g., `send.therunrec.com`), Klaviyo signs with their generic domain — bad for deliverability. |
| **Authentication** (DKIM / SPF / DMARC) | DNS records that prove your sender domain is legitimately you. Without these, Gmail/Yahoo will dump your emails to spam. |
| **Merge Tag** | A placeholder in copy that gets replaced with profile-specific data at send time. Example: `{{ first_name }}` becomes "Hassan" when the email goes to Hassan. |
| **Dashboard** | The deployed visual reference at https://saldader.github.io/runrec-email-pipeline/ that shows every email and SMS in the system. Each flow guide will tell you which dashboard URL to copy from. |
| **Buildsheet** | The architecture document at `~/vault/runrec/marketing/klaviyo-master-buildsheet-v2.md`. Reference for *why* things are designed the way they are. |
| **Voice Doctrine** | The rule for which voice signs each touch (Sal & Izzy / Sal alone / `{{location_manager}}` / The RunRec Team). See buildsheet Section 00.5. |
| **Mechanic Doctrine** | The rule for redemption mechanism (reply-to-redeem, not promo codes). See buildsheet Section 00.6. |

---

## Key contact

When stuck, ask **Sal (Faisal Dader)** — operator and decision-maker for everything in this package. Decisions that require his sign-off are flagged inline in the per-flow guides.

For technical work involving Skedda (booking system) integration or Stripe webhook setup, Sal will route to **Rob** or **Eyad** as needed.

---

## Final reminder

Every email you send goes to a real person who trusted RunRec with their inbox. The point of the structure in this package — the test protocols, the activation checklists, the don't-do list — is to keep that trust. Move slowly. When in doubt, send a test. When still in doubt, ask Sal.
