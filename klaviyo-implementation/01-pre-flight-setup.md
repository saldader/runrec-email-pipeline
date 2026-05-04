# 01 — Pre-Flight Setup

**Read this entire file before clicking anything.** This is the infrastructure work that has to be done in Klaviyo *before* you can build any of the 13 flows. None of the per-flow guides will work if Pre-Flight is skipped or partially done.

**Estimated time:** 6-10 hours of focused work, plus 1-3 days of waiting on DNS propagation (Section B), plus engineering coordination time (Sections G and parts of D).

**You will need:**
- Owner or Admin access to Klaviyo
- Access to therunrec.com DNS records (or coordination with Rob)
- Sal reachable for decisions flagged in this doc

---

## Section A — Account access verification

### A.1 Confirm your role

1. Log in to Klaviyo at **https://www.klaviyo.com/login**
2. In the top-right corner, click your initials (avatar circle) → **Account**
3. In the left sidebar, click **Users**
4. Find your name in the list. Look at the **Role** column.

You need **Owner** or **Admin** role. If you see anything else (Manager, Analyst, Campaign Coordinator, Content Creator), stop. Ask Sal to upgrade your role before continuing.

### A.2 Invite additional admins (if needed)

If anyone else needs admin access (Eyad, for example):

1. From the same **Users** page, click **Add new user** (top-right)
2. Enter their email address
3. Set role to **Admin**
4. Click **Add user**

They will get an email invitation. They must click the link and finish account setup, including 2FA, before they appear as active.

### A.3 Enable Two-Factor Authentication (2FA)

This is required for everyone who will touch the account.

1. Top-right initials → **My Account**
2. Click **Two-factor authentication** in left sidebar
3. Click **Enable**
4. Use an authenticator app (Google Authenticator, Authy, or 1Password). Avoid SMS-based 2FA — it's less secure.
5. Scan the QR code with your authenticator app
6. Enter the 6-digit code from the app to confirm
7. **Save the backup codes** somewhere safe (1Password recommended). If you lose your phone, these are the only way back in.

---

## Section B — Domain authentication (CRITICAL — do not skip)

This is the single most important infrastructure step. Without proper domain authentication, Gmail and Yahoo will dump RunRec emails into spam folders, and the open rates in this whole package will collapse.

### What this is, in plain language

Email providers (Gmail, Yahoo, Outlook) check whether the sender of an email is allowed to send on behalf of the domain in the From: address. If you don't prove it, they assume the email is impersonating someone and bury it.

Three records prove it:
- **SPF** — a list of services authorized to send email on behalf of `therunrec.com`
- **DKIM** — a cryptographic signature on every email proving it came from Klaviyo legitimately
- **DMARC** — a policy telling email providers what to do if SPF or DKIM fails (allow, quarantine, reject)

A fourth record (BIMI) shows RunRec logo next to the sender name in supported inboxes — nice-to-have, not required.

### B.1 Set up the branded sending domain

1. Top-right initials → **Account**
2. Left sidebar → **Domains**
3. You should see a section called **Custom Sending Domain**
4. Click **Set up custom sending domain**
5. Enter the subdomain you want Klaviyo to use. Recommended: `send.therunrec.com`
   - **Why a subdomain?** Keeps Klaviyo's reputation isolated from your main `therunrec.com` domain (used by your Google Workspace inbox). If something goes wrong with Klaviyo, your team's inboxes are unaffected.
6. Klaviyo will display a list of DNS records you need to add. Keep this page open.

### B.2 Add the DNS records

DNS records live with whoever runs your domain (probably your registrar — Namecheap, Cloudflare, GoDaddy, or whoever owns therunrec.com).

**If you don't have access to DNS yourself, this is where you stop and escalate to Sal or Rob.** Forward them the screenshot of the records Klaviyo is asking you to add. Engineering or domain admin needs to add them.

The records will look like this (example values — yours will be different):

| Type | Host | Value | TTL |
|------|------|-------|-----|
| CNAME | `s1._domainkey.send` | `s1.domainkey.uXXXXXXXXX.wlYYY.sendgrid.net` | 1 hour |
| CNAME | `s2._domainkey.send` | `s2.domainkey.uXXXXXXXXX.wlYYY.sendgrid.net` | 1 hour |
| CNAME | `em.send` | `uXXXXXXXXX.wlYYY.sendgrid.net` | 1 hour |
| TXT | `send` (or `_dmarc.send`) | `v=DMARC1; p=none; rua=mailto:dmarc@therunrec.com` | 1 hour |

Copy each one **exactly** as Klaviyo shows it. One typo and authentication fails.

### B.3 SPF record

If `therunrec.com` is already used to send email from other services (Google Workspace for sure, possibly others), the SPF record needs to **include Klaviyo** without removing the existing services.

1. Find your current SPF record. It will be a TXT record on `therunrec.com` starting with `v=spf1`. Example: `v=spf1 include:_spf.google.com ~all`
2. Add Klaviyo's SPF include before the `~all`. The new record should look like: `v=spf1 include:_spf.google.com include:_spf.klaviyo.com ~all`
3. **Do not create a second SPF record.** A domain can have only one SPF record. Update the existing one.

If this is too technical, hand it to Rob/engineering. SPF mistakes break sending for the entire domain (including Google Workspace inboxes).

### B.4 DMARC record

DMARC tells email providers what to do when SPF or DKIM fails.

Recommended starting policy: `v=DMARC1; p=none; rua=mailto:dmarc@therunrec.com`

- `p=none` means "monitor only — don't quarantine or reject anything yet."
- After 30 days of clean reports, advance to `p=quarantine` (suspect mail goes to spam).
- After another 30 days clean, advance to `p=reject` (suspect mail is bounced).

If `therunrec.com` already has a DMARC record (TXT record on `_dmarc.therunrec.com`), do not overwrite it without Sal's approval. Existing DMARC may already be set correctly.

### B.5 Wait for DNS propagation, then verify

DNS records take **15 minutes to 48 hours** to propagate. Usually 1-3 hours.

1. After ~2 hours, return to **Account → Domains** in Klaviyo
2. Click **Verify** on the custom sending domain
3. Each record will show a green checkmark or red X. If red, wait longer or check the record was entered correctly.
4. Once all records are green, the sending domain is authenticated.

### B.6 BIMI (optional, recommended)

BIMI displays your logo next to the sender name in Gmail and other supporting clients. Adds trust signal.

Requires:
- DMARC at `p=quarantine` or `p=reject` (so do this *after* B.4 is at quarantine)
- A SVG logo file at a publicly accessible URL
- A VMC (Verified Mark Certificate) — costs ~$1500/year from a certificate authority

This is optional. Skip until DMARC is at quarantine and you've validated the rest of the system. Sal makes the call on whether to invest in VMC.

### B.7 Verify deliverability

1. Klaviyo has a deliverability dashboard at **Account → Domains → Deliverability hub**
2. After authentication is complete, this dashboard should show "Authenticated" status with green indicators on SPF, DKIM, DMARC.
3. Klaviyo also runs spam scoring on your sender. Look for "Sender Reputation" — should be **Good** or **Excellent**.

**If sender reputation is Poor or Average:** Stop. Escalate to Sal. Do not start sending automated flow volume on a poor-reputation sender.

### B.8 Stop sign

If anything in Section B is unclear, blocked on DNS access, or returns errors that don't resolve in 48 hours: **stop and escalate to Sal**. Continuing without authentication is the single fastest way to wreck deliverability for the whole company.

---

## Section C — Sender profiles

A "sender profile" is the from-name and from-email shown to recipients. RunRec uses two.

### C.1 Create the consumer sender (`contact@therunrec.com`)

1. **Account → Settings → Email → Sender profiles**
2. Click **Add sender profile**
3. Configure:
   - **From email:** `contact@therunrec.com`
   - **From label (name shown to recipient):** `RunRec`
   - **Reply-to email:** `contact@therunrec.com` (same as from)
4. Click **Save**
5. Klaviyo will send a confirmation email to `contact@therunrec.com`. Someone with access to that inbox needs to click the verification link.

This is the default sender for almost every flow (Welcome, Booking, Post-Session, Birthday, Anniversary, Win-Back, VIP, Lead Nurture, Membership, Abandoned Booking, Sunset).

### C.2 Create the corporate sender (`{{location_manager}}@therunrec.com`)

This is the tricky one. The Voice Doctrine says corporate flow (Section 9) is sent from the **local location manager** (e.g., `eyad@therunrec.com` for Waterloo). But Klaviyo doesn't let you put a merge tag in the From: field — it has to be a literal email address that's been verified.

**Operator decision needed (Sal):** Choose one of two options.

**Option A — Multi-location-correct (recommended):** Create a separate sender profile per location. For Waterloo, create `eyad@therunrec.com` (or whoever is the Waterloo location manager). When future franchisees clone, they create their own. Section 9 flow gets cloned per location with the correct sender hardcoded.

**Option B — Fallback (single sender, merge tag in body):** Use a single sender like `team@therunrec.com` for all corporate sends. Use `{{ location_manager }}` in the email body and signature only. Simpler to maintain, but the From: header won't match the body's signature, which the audit flagged as an issue.

**Default (until Sal decides):** Build with Option A for Waterloo. Create the sender `eyad@therunrec.com` (or correct manager email). Document the requirement for future franchises.

To create it:

1. **Account → Settings → Email → Sender profiles → Add sender profile**
2. **From email:** `eyad@therunrec.com` (or whatever the Waterloo manager email is)
3. **From label:** Use the manager's actual name, e.g., `Eyad at RunRec`
4. **Reply-to:** Same as from
5. **Save**, then verify the address from that inbox.

### C.3 Verify both senders work

After both are verified, send yourself a test campaign from each sender (don't actually send to a list — use the **Send Test** function inside any draft campaign). Confirm they arrive in your inbox with the correct From: name and email.

---

## Section D — Custom profile properties (REQUIRED)

A profile property is a piece of data attached to each customer record. Klaviyo comes with built-ins (first name, email, phone) but the 13 flows reference **17+ custom properties** that don't exist yet.

### D.1 How to create a custom profile property

1. **Account → Settings → Profile properties**
2. Click **Add new property**
3. Enter:
   - **Property name:** Use the exact name from the table below (case-sensitive — `last_booking_date`, not `Last Booking Date` or `lastBookingDate`)
   - **Property type:** Choose Text, Number, Date, or Boolean per the table
4. Click **Save**

Repeat for every property below. There are roughly 22 of them. Allow ~30 minutes.

### D.2 Properties to create

These are listed in priority order. Properties marked **HIGH** are required for the highest-priority flows; do these first.

#### Standard / built-in (verify these already exist)

| Property | Type | Notes |
|----------|------|-------|
| `first_name` | Text | Standard Klaviyo field — already exists. Verify populated for most profiles. |
| `last_name` | Text | Standard Klaviyo field — already exists. Verify populated for most profiles. |
| `email` | Text | Standard Klaviyo identifier. |
| `phone_number` | Text | Standard Klaviyo identifier. E.164 format (e.g., `+15551234567`). |

#### Location / sender (HIGH — required for franchise readiness)

| Property | Type | Source / how it gets populated |
|----------|------|---------------------------------|
| `location` | Text | Manual (admin assigns by which RunRec the customer books at). Future: auto-set by Skedda webhook. Values: `Waterloo`, `Burlington`, etc. |
| `location_name` | Text | Same as `location` — used in body copy where the brand name needs to render. Today: just "RunRec" everywhere. |
| `location_manager` | Text | Manual. The local manager's first name. Example: `Eyad`. |
| `location_manager_email` | Text | Manual. The local manager's email. Example: `eyad@therunrec.com`. |
| `location_phone` | Text | Manual. The local facility phone. Example: `+15191234567`. |

#### Booking metadata (HIGH — Flows 02, 03, 06, 12)

| Property | Type | Source |
|----------|------|--------|
| `last_booking_date` | Date | Skedda webhook → custom event → profile update. **Engineering work needed.** |
| `last_sport_booked` | Text | Skedda webhook. Values: `Basketball`, `Volleyball`, `Pickleball`, `Badminton`, `Dodgeball`. |
| `booking_count` | Number | Skedda webhook (incremented on each `booking_completed` event). |
| `total_spend` | Number | Stripe webhook (sum of all Successfully Paid events for the profile). |
| `court_letter` | Text | Skedda webhook (for booking confirms). Values: `A`, `B`, `C`. **Note:** the audit flagged that "Court A" naming is deprecated; Sal to confirm whether to use court letter or facility section name. |
| `last_review_disposition` | Text | Custom — set by Flow 03 NPS thumbs response. Values: `positive`, `negative`, `unknown`. |

#### Birthday / family (HIGH for Flow 04, 05)

| Property | Type | Source |
|----------|------|--------|
| `dob` | Date | Manual or signup form. Format: full date (YYYY-MM-DD). Used by Flow 04 to fire 7 days before, on the day, and 1 day after birthday. |
| `child_name` | Text | Birthday party booking form. Used in Flow 05 personalization. |
| `child_age` | Number | Birthday party booking form. Optional — Flow 05 designed to work without. |
| `last_party_date` | Date | Manual or party-booking system. Used by Flow 05 to fire 11 months later. |
| `bday_gift_redeemed_year` | Number | Custom — set when Flow 04 birthday gift is redeemed. Prevents double-gifting same year. Values: `2026`, `2027`, etc. |

#### Membership / VIP (HIGH for Flows 07, 10, 11)

| Property | Type | Source |
|----------|------|--------|
| `vip_since` | Date | Custom — set when profile first hits VIP segment criteria (5+ bookings in 90 days). |
| `membership_tier` | Text | Stripe subscription webhook. Values: `None`, `Common User`, `Tri-City Member`, `VIP`. |
| `membership_started_date` | Date | Stripe `customer.subscription.created` event. **Engineering work — Stripe integration scope needs extending.** |
| `last_membership_renewal` | Date | Stripe `invoice.payment_succeeded` (recurring payments). |
| `membership_status` | Text | Stripe webhook. Values: `active`, `cancelled`, `lapsed`, `none`. |

#### Corporate / B2B (HIGH for Flow 09)

| Property | Type | Source |
|----------|------|--------|
| `corporate_inquiry_date` | Date | Website B2B form → Klaviyo event → profile update. |
| `corporate_status` | Text | Manual (sales/Sal updates). Values: `Lead`, `Active`, `Lapsed`, `Lost`. |
| `corporate_company_name` | Text | Website B2B form. |
| `corporate_event_size` | Number | Website B2B form (estimated headcount). |

#### Offer / abandoned booking (Flow 12)

| Property | Type | Source |
|----------|------|--------|
| `expiry_date` | Date | Custom — set when an offer is sent, marks when it expires. |
| `similar_booking_1`, `similar_booking_2`, `similar_booking_3` | Text | Set at abandoned-booking time — recent similar slots the customer might book instead. Three separate properties. |

### D.3 Verify properties were created

1. **Account → Settings → Profile properties**
2. Scroll the list. Every property in D.2 should be present.
3. Click any property to view its detail page. Confirm name and type are correct.

### D.4 Document who populates what

For each property, you (the admin) need to know how it gets data. Many of these depend on engineering work that hasn't happened yet. Use the source column above as your map. Anything that says **engineering work needed** has to wait until that work is done — flag to Sal.

---

## Section E — Lists architecture

### E.1 List vs Segment — the difference (READ THIS)

This trips up almost every new Klaviyo user. The two are not the same.

- **List:** A static collection. Someone is on it because they were added (signed up, imported, manually added). They stay on it until they're removed. Lists are often used as the "front door" — when someone signs up at runrec.com, they go on the Newsletter list.
- **Segment:** A dynamic collection. Someone is in it because they currently match a rule. Klaviyo updates segment membership automatically — if they stop matching, they're out; if they start matching, they're in. Segments are the "filters" you target campaigns and flows with.

**Rule of thumb:** Use Lists for explicit signups (Newsletter, Member roster). Use Segments for behavioral targeting (Engaged in 30 days, Lapsed bookers).

### E.2 Lists to keep (already exist)

The current Klaviyo account has 12 lists. The buildsheet specifies which to keep, archive, and how to use them. Decisions are already made — your job is to verify and clean up.

| List | List ID | Action | Notes |
|------|---------|--------|-------|
| Newsletter | SDFmeU | Keep — convert to double opt-in (see E.3) | The main marketing list. ~1,435 profiles. |
| Skedda Email List | WyA26L | Keep | 767 booking customers. Currently no flow attached. Will become the warm pool for win-back and post-session flows. |
| Tri-City Member Emails | XbJSh7 | Keep | 63 members. Will be wired to Flow 11 (Membership Lifecycle). |
| Birthday Customers | T8HDuV | Keep | 19 birthday party customers. Will be wired to Flows 04 + 05. |
| SMS Recipients | UjrHmK | Keep | 38 SMS-consented profiles. |
| Previous Campers | RVpxwS | Keep | 109 camp alumni. Used for Flow 06 (Win-Back) targeting. |
| PlayForever Signups | TS6RMf | Keep — no flow attached | Active partnership. Used for occasional sends only. |
| PlayForever League Emails | XHBsJD | Keep — no flow attached | Same. |
| Principle Emails | SQQBdj | Keep — quarantine | Scraped contacts. **CASL FLAG** — see compliance file. Do not send marketing without express consent. |

### E.3 Convert Newsletter to double opt-in

Currently, the Newsletter list is single opt-in. Best practice is double opt-in (signup → confirmation email → click confirm → on the list).

1. **Audience → Lists & segments**
2. Click on **Newsletter** list
3. Click **Settings** (top right)
4. Find **Opt-in Process**. Change from "Single opt-in" to "Double opt-in."
5. Klaviyo will offer a confirmation email template. Use defaults or customize.
6. **Save**

**Important:** This only affects new signups. Existing 1,435 profiles stay subscribed. They are grandfathered as legacy implied consent.

### E.4 Archive these lists

| List | List ID | Why |
|------|---------|-----|
| Camp Waitlist (Summer 2025) | ULfg93 | Past event, 9+ months stale |
| Jr. NBA Emails | StczEH | Program discontinued |
| Preview Test (Our Email) | Y2WZLm | Internal — not needed |

To archive a list:

1. **Audience → Lists & segments**
2. Hover over the list → click the three-dot menu
3. **Delete list**
4. Confirm. (Klaviyo doesn't have an "archive" — only delete. The profiles in those lists are NOT deleted, only removed from the list.)

**Sal decision needed:** Some teams prefer renaming (prefix `[ARCHIVED]`) instead of deleting. Either works. Default: delete to keep the workspace clean.

### E.5 New lists to create

| List name | Opt-in process | Purpose | Created by |
|-----------|----------------|---------|------------|
| `Members - VIP` | Double | VIP/Power User customers. Triggers Flow 07. Populated by automation when profile hits VIP segment criteria. | Manual now; later automation. |
| `Corporate Leads` | Double | B2B leads from website form. Triggers Flow 09. | Website form integration. |
| `Sunset Candidates` | Double | Profiles in their final 14 days before suppression. Triggers Flow 13. | Automation from Sunset segment. |

To create a list:

1. **Audience → Lists & segments → Create list / segment**
2. Choose **List**
3. Name it exactly as shown above
4. Set opt-in process: **Double opt-in**
5. Click **Create list**

---

## Section F — Segments

You'll create 13 segments. They take roughly 5 minutes each. Plan ~75 minutes.

### F.1 How to create a segment in Klaviyo

1. **Audience → Lists & segments → Create list / segment**
2. Choose **Segment**
3. Name it exactly as shown in F.2 below
4. Build the definition using the rule builder:
   - **What someone has done (or not done):** for behavior-based rules (opens, clicks, bookings)
   - **Properties about someone:** for profile-property-based rules (location, membership_status, dob)
   - **If someone is or is not in a list:** for list-based rules
5. Add multiple conditions with **AND** (must match all) or **OR** (must match any)
6. Click **Create segment**
7. Klaviyo calculates members. May take a few minutes for the count to populate.

### F.2 Segments to create

#### Tier 1 — Engagement (build first)

| Segment | Definition | Used by |
|---------|-----------|---------|
| `Engaged - 30 Days` | What someone has done: Opened Email OR Clicked Email — at least once — in the last 30 days | Most campaign sends; flow filters |
| `Engaged - 60 Days` | What someone has done: Opened Email OR Clicked Email — at least once — in the last 60 days | Broader campaign sends |
| `Engaged - 90 Days` | What someone has done: Opened Email OR Clicked Email — at least once — in the last 90 days | Re-engagement; broad sends |
| `Unengaged - 90+ Days` | What someone has done: Received Email — at least once — in the last 365 days AND Opened Email — zero times — in the last 90 days AND Clicked Email — zero times — in the last 90 days | Flow 13 Sunset trigger |

#### Tier 2 — Customer type

| Segment | Definition | Used by |
|---------|-----------|---------|
| `Court Bookers` | What someone has done: Successfully Paid — at least once — in the last 365 days | Flow 03, 06 filtering |
| `Members - Active` | Properties: `membership_status` equals `active` | Flow 07 inclusion; Flow 11 |
| `Members - Lapsed` | Properties: `membership_status` equals `cancelled` OR `lapsed` | Flow 11 win-back path |
| `Never Booked` | What someone has done: Successfully Paid — zero times — over all time AND If someone is or is not in a list: is in `Newsletter` | Flow 08 Lead Nurture trigger |
| `New Subscribers` | If someone is or is not in a list: is in `Newsletter` (joined in the last 14 days) | (Already exists in account — verify) |

#### Tier 3 — Behavioral

| Segment | Definition | Used by |
|---------|-----------|---------|
| `First-Time Booker` | What someone has done: Successfully Paid — exactly 1 time — over all time | Post-purchase split |
| `Repeat Booker` | What someone has done: Successfully Paid — at least 2 times AND at most 4 times — over all time | Welcome split, Win-Back filtering |
| `VIP / Power User` | What someone has done: Successfully Paid — at least 5 times — in the last 90 days | Flow 07 trigger; Flow 10 inclusion |
| `Lapsed Bookers` | Properties: `last_booking_date` is more than 60 days ago AND What someone has done: Successfully Paid — at least 1 time — over all time | Flow 06 Win-Back trigger |

### F.3 Verify segments populated

After creating each, give Klaviyo 15-30 minutes to calculate. Then click each segment and confirm:
- Member count is non-zero (assuming the property data exists)
- Sample of members in the segment looks correct (open a profile and verify)

If a segment shows zero members and you expected some, the most common cause is the underlying property hasn't been populated yet. Engineering work (Section G) usually has to land first.

---

## Section G — Custom events from external systems

The plan needs **8 custom events** that don't currently exist in Klaviyo. These come from outside systems (Skedda for bookings, Stripe for subscriptions, your website form for corporate inquiries). Each one requires a webhook from the source system to Klaviyo's API.

**This is engineering work.** As the admin, your job is to:
1. Document what's needed
2. Coordinate with Eyad / Rob to build the webhooks
3. Verify events appear in Klaviyo's metric list once built

You should not attempt to build webhooks yourself unless you're comfortable with API work.

### G.1 Events needed

| Event name | Source system | Triggered by | Used by flow | Status |
|------------|--------------|--------------|--------------|--------|
| `booking_created` | Skedda | Customer completes a booking | Flow 02 (Booking Confirm) | **Missing — Skedda doesn't fire any events to Klaviyo today** |
| `booking_completed` | Skedda | 2 hours after booking end-time | Flow 03 (Post-Session) | Missing — proxy is currently `Successfully Paid` from Stripe, which is wrong (paid ≠ attended) |
| `booking_abandoned` | Vercel checkout layer | Customer reaches checkout but doesn't pay within 30 min | Flow 12 (Abandoned Booking) | Missing — flagged in buildsheet as Eyad/Rob action item |
| `corporate_inquiry` | Website form | B2B contact form submitted | Flow 09 (Corporate) | Missing — needs website form + Klaviyo Forms event |
| `membership_started` | Stripe | `customer.subscription.created` webhook | Flow 11 (Lifecycle) | Missing — Stripe integration is paid-events-only currently |
| `membership_renewed` | Stripe | `invoice.payment_succeeded` (recurring) | Flow 11 | Missing — same Stripe scope expansion |
| `membership_cancelled` | Stripe | `customer.subscription.deleted` | Flow 11 win-back | Missing — same Stripe scope expansion |
| `birthday_party_completed` | Manual or Skedda | Day after a birthday party | Flow 05 (Anniversary) | Missing — likely manual data entry today |

### G.2 Coordinating the work

For each event, the engineer (Eyad / Rob) needs:
1. The event name (use exactly the names in the table)
2. The expected payload (what data to send to Klaviyo)
3. The Klaviyo private API key (Sal can generate from Account → Settings → API keys — never share publicly)
4. Klaviyo's events API endpoint: `https://a.klaviyo.com/api/events/`

The standard payload structure looks like:

```json
{
  "data": {
    "type": "event",
    "attributes": {
      "metric": { "data": { "type": "metric", "attributes": { "name": "booking_created" } } },
      "profile": { "data": { "type": "profile", "attributes": { "email": "customer@example.com" } } },
      "properties": {
        "booking_id": "abc123",
        "court_letter": "A",
        "sport": "Basketball",
        "booking_date": "2026-05-15",
        "booking_time": "19:00"
      }
    }
  }
}
```

The exact properties for each event are specified in the per-flow guides (where each flow tells you what data it needs from the trigger event).

### G.3 Verify events appear in Klaviyo

After engineering builds a webhook, send a test trigger and verify:

1. **Analytics → Metrics**
2. Search for the event name (e.g., `booking_created`)
3. The metric should appear in the list. Click it.
4. You should see at least one event firing in the last 24 hours.
5. Click an individual event to inspect the payload — confirm the properties you expected are there.

If the event doesn't appear, engineering issue. Escalate.

---

## Section H — Templates

The current account has 21 templates, none of them RunRec-branded. They're all 2022-era Klaviyo defaults. You need to build **3 base templates** for the flow library to use.

### H.1 General template-build rules

For all templates:
- Mobile-first (75%+ of opens are mobile)
- 600px max width, single column
- 60/40 text-to-image ratio minimum
- All assets hosted on therunrec.com or Klaviyo CDN
- Footer must include: business name (`RunRec`), physical address (`283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8`), unsubscribe link, "manage preferences" link
- Plain text version always included (Klaviyo auto-generates; verify)

### H.2 How to create a template from scratch

1. **Content → Templates**
2. Click **Create template** (top-right)
3. Choose **Drag-and-drop** (easier for non-technical) or **HTML** (full control, requires code)
4. Name it exactly as below
5. Build the design (instructions below per template)
6. Click **Save**

### H.3 Template 1 — Flow / Personal (used by 10 of 13 flows)

**Name:** `RR - Flow Personal v1`

**Look:** Minimalist. Looks like a personal email from a real person, not marketing.

**Layout:**
- No header image
- Single column
- Logo at the very top (small, ~120px wide), left-aligned
- Body text (single text block, customizable per email)
- One CTA button or one inline link maximum
- PS line at the bottom (above footer)
- Footer: small grey text — Business name, physical address, unsubscribe link, manage preferences link

**Used by:** Welcome (Flow 01), Booking Confirm (02), Post-Session (03), Birthday (04), Win-Back (06), Lead Nurture (08), Corporate (09), Membership Lifecycle (11), Abandoned Booking (12), Sunset (13)

### H.4 Template 2 — Birthday Party (used by Flow 05)

**Name:** `RR - Birthday Party v1`

**Look:** Branded, warm, party-themed. More visual than the Flow Personal template.

**Layout:**
- Header image (party photo, optional — provided by Sal)
- Single column
- Logo
- Body text
- One CTA button
- Footer same as Flow Personal

**Used by:** Birthday Party Anniversary (Flow 05)

### H.5 Template 3 — VIP / Status (used by Flow 07 only)

**Name:** `RR - VIP Status v1`

**Look:** Premium feel. Dark accent color. Designed to feel exclusive.

**Layout:**
- Top accent bar (dark — black or RunRec brand dark color)
- Logo (white version, on dark background)
- Body text
- Status indicator (badge or text element — "VIP," "Power User," etc.)
- Single CTA
- Footer same as Flow Personal

**Used by:** VIP + Referral (Flow 07)

### H.6 Verify templates

After building each:

1. **Content → Templates → click on each template**
2. Click **Preview** — confirm desktop and mobile views look correct
3. Send a test to yourself — confirm it renders in your inbox

### H.7 Delete or archive old templates

Per the audit, all 21 existing templates can be archived/deleted. They're 2022 starter templates. To delete:

1. **Content → Templates**
2. Hover over template → three-dot menu → **Delete**
3. Confirm

**Sal decision needed:** Some operators prefer to keep one Klaviyo default as a reference. Either approach is fine.

---

## Section I — Klaviyo account settings

A few global settings to verify.

### I.1 Smart Sending

**What it is:** Klaviyo prevents the same person from getting the same email or SMS within a short time window. Default: 16 hours for email, 12 hours for SMS.

**Why it matters:** If a customer is in two flows that both want to email them on the same day, Smart Sending makes sure they only get one of the two.

**To enable / verify:**

1. **Account → Settings → Email** (or **SMS**)
2. Find **Smart Sending**
3. Confirm it's **enabled**
4. Email window: **16 hours** (default)
5. SMS window: **12 hours** (default)
6. **Save**

### I.2 Quiet Hours

**What it is:** Klaviyo will hold sends and queue them to deliver inside the specified hours, per the recipient's timezone.

**Standard:** 9 PM to 9 AM (do not send during these hours).

**To set:**

1. **Account → Settings → SMS** (most important for SMS)
2. Find **Quiet Hours**
3. Set: **From 9 PM to 9 AM** (recipient local time)
4. **Save**
5. Repeat under **Account → Settings → Email** (less critical for email but recommended)

### I.3 Bounce handling

**What it is:** When emails bounce (the address doesn't exist or is full), Klaviyo decides whether to suppress the profile or keep trying.

**Klaviyo default:** Suppress after 7 soft bounces. The CMO skill recommends 2-3 for tighter list hygiene.

**To configure:**

1. **Account → Settings → Email → Deliverability**
2. Find **Bounce thresholds**
3. Adjust **Soft bounce limit** to **3**
4. **Save**

### I.4 Spam complaint handling

Klaviyo auto-suppresses profiles who hit "Mark as Spam." Verify this is on.

1. **Account → Settings → Email → Deliverability**
2. Confirm **Spam complaint auto-suppression** is **enabled**

---

## Section J — Existing flow cleanup

Before building new flows, the conflicting live flows in the account need to be paused or modified. **Do not delete them** — pausing preserves the existing data and lets you reactivate if needed.

### J.1 Why this matters

Per the inventory, **4 flows are live** that overlap with the planned new flows:

1. `RZsSVX` — 2025 Welcome Flow (Visual Email) — **KEEP, will be extended in Step 6 of the build order**
2. `RncQfn` — 2025 Browse Abandoned (Visual Email) — **PAUSE NOW** (fires on `Viewed Product` which is high-volume / low-intent — risk of over-mailing)
3. `WajxZ4` — 2025 Browse Abandoned (SMS Part Flow 3) — **PAUSE NOW** (same reason)
4. `VwYTDc` — 2025 Post Purchase (Visual Email) — **KEEP for now**, will be replaced when Flow 03 (Post-Session) is built and Skedda integration lands
5. `YvvKXF` — Win Back (Email Flow Part 5) — **KEEP, will be extended in Step 7 of the build order**

### J.2 How to safely pause a flow

1. **Flows → click on the flow name**
2. Top-right: change status from **Live** to **Manual**
3. Confirm

**What happens when you pause:** New profiles will not enter the flow. Profiles currently mid-flow continue receiving the rest of the flow as scheduled (they don't get orphaned).

### J.3 Flows to pause now (before any new flows go live)

| Flow | ID | Action | Reason |
|------|----|--------|--------|
| 2025 - Browse Abandoned (Visual Email) | RncQfn | Pause (set to Manual) | Fires on Viewed Product. Will be replaced by Flow 12 when `booking_abandoned` event lands. |
| 2025 - Browse Abandoned (SMS Part Flow 3) | WajxZ4 | Pause (set to Manual) | Same as above. |

### J.4 Flows to archive (drafts, no impact)

These are draft stubs that were never finished. Safe to archive.

| Flow | ID |
|------|-----|
| Draft - Abandoned Cart (Email Flow Part 2) | U25Ffk |
| Draft - Abandoned Cart (SMS Flow Part 2) | R3f8r8 |
| New - Browse Abandoned (Email Flow Part 3) | TTiUHs |
| New - Post Purchase (Email Flow Part 4) | VvLxsN |
| Draft - Win Back (SMS Flow Part 5) | XHEScZ |
| Essential Flow Recommendation_ | VxHPF8 |
| Essential Flow Recommendation_ | WH8EyY |

To archive: **Flows → click flow → ⋯ menu → Archive**.

### J.5 Sal decision: First Purchase Anniversary flow

Flow `UBQq2E` (First Purchase Anniversary - Standard) has been live since 2022. Anyone who paid in the last 4 years is in this flow's queue.

- **Option A:** Keep it. Continue to overlap with Flow 11 (Membership Lifecycle).
- **Option B:** Pause it. Profiles mid-queue lose their anniversary email.
- **Option C:** Re-engineer it as part of Flow 11.

Default until Sal decides: **leave it live**. Flag for follow-up.

---

## Section K — Pre-Flight Checklist

Before moving on to building flows (`02-build-template.md` and the per-flow guides), every item below must be **green**. This is your gate.

### Account & access
- [ ] You are logged in as Owner or Admin (Section A.1)
- [ ] 2FA is enabled on your account (Section A.3)
- [ ] Sal and any required teammates have admin access (Section A.2)

### Domain authentication
- [ ] Custom sending domain set up (`send.therunrec.com` or chosen subdomain) (Section B.1)
- [ ] DNS records (SPF, DKIM, DMARC) added (Section B.2-B.4)
- [ ] All records show green checkmark in Klaviyo (Section B.5)
- [ ] Sender reputation is **Good** or **Excellent** (Section B.7)
- [ ] BIMI deferred until DMARC at `p=quarantine` (Section B.6) — optional

### Sender profiles
- [ ] `contact@therunrec.com` sender created and verified (Section C.1)
- [ ] Corporate sender (e.g., `eyad@therunrec.com`) created and verified (Section C.2)
- [ ] Test email from each sender arrived in your inbox correctly (Section C.3)

### Custom profile properties
- [ ] All ~22 properties from Section D.2 created
- [ ] Property names match exactly (case-sensitive)
- [ ] Property types correct (Text/Number/Date/Boolean)
- [ ] Source for each documented (Section D.4)

### Lists
- [ ] Newsletter converted to double opt-in (Section E.3)
- [ ] Old lists archived (Camp Waitlist, Jr. NBA, Preview Test) (Section E.4)
- [ ] New lists created (Members - VIP, Corporate Leads, Sunset Candidates) (Section E.5)

### Segments
- [ ] All 13 segments from F.2 created
- [ ] Each segment shows non-zero population (where data exists) within 30 minutes (Section F.3)

### Custom events (engineering)
- [ ] Engineering team aware of the 8 events needed (Section G.1)
- [ ] Booking events (`booking_created`, `booking_completed`, `booking_abandoned`) — engineering work scheduled
- [ ] Stripe subscription events scope expansion — engineering work scheduled
- [ ] Corporate inquiry form + Klaviyo Forms wiring — engineering work scheduled
- [ ] When events are wired, verify in Analytics → Metrics (Section G.3)

### Templates
- [ ] `RR - Flow Personal v1` template built and tested (Section H.3)
- [ ] `RR - Birthday Party v1` template built and tested (Section H.4)
- [ ] `RR - VIP Status v1` template built and tested (Section H.5)
- [ ] All include unsubscribe link, physical address, business name in footer (Section H.1)
- [ ] Old default Klaviyo templates archived/deleted (Section H.7)

### Account settings
- [ ] Smart Sending enabled (Section I.1)
- [ ] Quiet Hours set to 9 PM - 9 AM recipient timezone (Section I.2)
- [ ] Bounce threshold set to 3 soft bounces (Section I.3)
- [ ] Spam complaint auto-suppression enabled (Section I.4)

### Flow cleanup
- [ ] Browse Abandoned flows (`RncQfn`, `WajxZ4`) paused (Section J.3)
- [ ] Draft stubs archived (Section J.4)
- [ ] Sal aware of First Purchase Anniversary decision pending (Section J.5)

---

## When the checklist is green

You are ready to move to `02-build-template.md`. Open it and read it all the way through before opening the first per-flow guide.

If you hit blockers — engineering work not done, Sal hasn't decided something, DNS issues persist — **do not start building flows on top of incomplete infrastructure**. Stop and resolve. Half-built flows on a half-built foundation produce broken sends, and broken sends erode the warm subscriber base you spent years building.
