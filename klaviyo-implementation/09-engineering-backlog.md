# Engineering Backlog — Required for Klaviyo Plug-and-Play

**For:** Eyad / Rob / engineering owner
**Status:** Open — these tickets must be resolved before specific flows can launch

This is the integration work that's NOT visible in the Klaviyo UI but blocks the plug-and-play package. Each ticket has: blocker scope, what to build, definition of done, and which flow it unblocks.

---

## Ticket-Ready Backlog

### TICKET 1 — Skedda → Klaviyo: Booking Created event

**Blocks:** Flow 02 (Booking Confirm + Reminders), Flow 03 (Post-Session + Review)
**Severity:** Critical (without this, 2 of 13 flows can't launch)

**What to build:**
- Webhook from Skedda's "booking created" event to Klaviyo's Track API
- Event name in Klaviyo: `Booking Created`
- Required event payload properties:
  - `customer_email` (string) — used to identify the profile
  - `booking_id` (string) — unique
  - `court_letter` (string — A/B/C)
  - `sport` (string — Basketball/Volleyball/Pickleball/Dodgeball)
  - `booking_date` (ISO date)
  - `booking_start_time` (ISO datetime)
  - `booking_end_time` (ISO datetime)
  - `event_type` (string — "court_booking" / "membership_renewal" / "party_deposit") — needed to filter what fires what flow
  - `location` (string — Waterloo / Burlington / etc.)
  - `total_paid` (number)
- Receiver: Klaviyo Track API endpoint, public key from Sal's Klaviyo settings

**Definition of done:**
- Test booking in Skedda fires the event into Klaviyo within 10 seconds
- Event appears on customer profile timeline
- Profile property `last_booking_date` updates automatically
- Profile property `booking_count` increments

**Reference:** Klaviyo Track API docs — https://developers.klaviyo.com/en/reference/track_event

---

### TICKET 2 — Skedda → Klaviyo: Booking Completed event

**Blocks:** Flow 03 (Post-Session) Touch 01 — fires X hours after session end
**Severity:** High

**What to build:**
- Webhook from Skedda's "booking completed" trigger (90 minutes after `booking_end_time`)
- Event name: `Booking Completed`
- Same payload as Booking Created, plus `completed_at` timestamp

**Definition of done:**
- Event fires reliably 90 minutes after the booking's actual end time
- Klaviyo can read `event_type` to filter (only court bookings get NPS, parties go to anniversary flow)

---

### TICKET 3 — Vercel checkout → Klaviyo: Booking Abandoned event

**Blocks:** Flow 12 (Abandoned Booking)
**Severity:** Critical

**What to build:**
- Track abandoned bookings from RunRec's Vercel-hosted checkout page
- Event name: `Booking Abandoned`
- Required event payload:
  - `customer_email` (or `customer_phone` for SMS-only)
  - `sport` (string)
  - `court_letter` (string)
  - `booking_date` (ISO date)
  - `booking_time` (ISO datetime)
  - `total_value` (number)
  - `resume_url` (string — pre-filled checkout URL)
  - `similar_booking_1`, `_2`, `_3` (string array — for the social proof in Touch 02)
- Trigger: customer reaches checkout, enters email, doesn't complete payment within 30 minutes

**Definition of done:**
- Klaviyo profile gets the event with all 9 properties populated
- `resume_url` actually works (clicking it returns to a pre-filled checkout)
- Test: abandon a booking → Flow 12 Touch 01 fires within 30 min

---

### TICKET 4 — Stripe → Klaviyo: Subscription lifecycle events

**Blocks:** Flow 11 (Membership Lifecycle) — all touches depend on subscription state changes
**Severity:** Critical

**What to build:**
- Currently Stripe → Klaviyo only syncs `Successfully Paid` (one-time payments)
- Extend integration to sync these subscription events:
  - `Subscription Created` → fires Flow 11 Touch 01 (Welcome to family)
  - `Subscription Updated` (status change to active) — for plan upgrades
  - `Subscription Cancelled` → fires Flow 11 cancellation branch (C1/C2)
  - `Subscription Renewed` (invoice.paid for subscription) → fires Flow 11 Touch 04 (Year in numbers) on the anniversary
- Required event properties: `subscription_id`, `plan_name`, `started_date`, `current_period_end`, `cancellation_reason` (if applicable)
- Profile property updates: `membership_tier`, `membership_status`, `membership_started_date`, `last_membership_renewal`

**Definition of done:**
- Stripe subscription test event triggers the corresponding Klaviyo event
- Profile properties update automatically
- Flow 11 fires on subscription created (verified in Klaviyo activity log)

---

### TICKET 5 — Website inquiry form → Klaviyo: Corporate Inquiry event

**Blocks:** Flow 09 (Corporate & League Nurture)
**Severity:** Critical

**What to build:**
- Build a corporate inquiry form on the RunRec website (or use the existing one if already built)
- Form fields: company name, contact name, contact email, contact phone, # of people, use case (team builder / league / corporate event), preferred location, preferred date range
- On submit: fire `Corporate Inquiry` event to Klaviyo
- Required event payload:
  - `company_name`
  - `contact_email`
  - `contact_phone`
  - `headcount`
  - `use_case`
  - `preferred_location`
  - `preferred_date_range`
  - `inquiry_date` (auto)
- Profile property updates: `corporate_status` = "Lead", `corporate_inquiry_date` = today

**Definition of done:**
- Form submission creates Klaviyo profile (if doesn't exist) or updates existing
- Event fires within 5 seconds of submission
- Flow 09 fires automatically on inquiry

---

### TICKET 6 — NPS thumbs response webhook

**Blocks:** Flow 03 (Post-Session) Touch 02-04 conditional branching on 👍 vs 👎 reply
**Severity:** Medium (Flow 03 can launch without this in degraded mode — send Touch 02 to everyone instead of branching)

**What to build:**
- SMS inbound webhook that detects emoji-only replies (👍 or 👎) and routes to one of two Klaviyo events:
  - `NPS Positive` (👍 reply received)
  - `NPS Negative` (👎 reply received)
- Service options: Twilio Functions, Klaviyo's SMS reply parser, custom AWS Lambda
- Required event payload: `customer_phone`, `original_send_id`, `response_emoji`

**Definition of done:**
- Test SMS reply with 👍 → `NPS Positive` event fires within 30 seconds
- Test SMS reply with 👎 → `NPS Negative` event fires
- Flow 03 conditional split routes correctly

**Fallback if not built:** Flow 03 Touch 02 (Google review nudge) sends to everyone regardless. Flag for retry on a Touch 03 send instead.

---

### TICKET 7 — Birthday party completion event

**Blocks:** Flow 05 (Birthday Party Anniversary)
**Severity:** Medium (Flow 05 only fires once per year per profile — can be manually loaded for v1)

**What to build:**
- Tag birthday party bookings in Skedda with an `event_type` of `birthday_party`
- When the booking completes (90 min after `booking_end_time`), fire `Birthday Party Completed` event
- Required event payload: `customer_email`, `child_name`, `child_age`, `party_date`, `package_tier` (HERO / SUPER HERO / SUPER HERO FUN), `headcount`
- Profile property updates: `last_party_date`, `child_name`, `child_age`

**Definition of done:**
- Birthday party booking marked correctly in Skedda
- Event fires after completion
- Profile properties populated

**Fallback for v1:** Quarterly manual export of birthday party customers from Skedda → manual upsert into Klaviyo with the properties set. Flow 05 trigger uses `last_party_date` property change instead of an event.

---

### TICKET 8 — Domain authentication: DKIM, SPF, DMARC

**Blocks:** EVERY flow — without this, deliverability tanks
**Severity:** Critical

**What to build:**
- Configure DKIM record for `therunrec.com` per Klaviyo's instructions (Settings → Domains → Setup)
- Verify SPF record includes Klaviyo's sending IP range
- Configure DMARC record (start with `p=none` for monitoring, escalate to `p=quarantine` after 30 days clean)
- (Optional but recommended) BIMI record for brand logo in Gmail

**Definition of done:**
- Klaviyo's domain authentication dashboard shows ALL GREEN
- DMARC reports flowing to a monitoring inbox
- Test sends from Klaviyo land in inbox (not spam) for Gmail / Outlook / Apple Mail / Yahoo
- 95%+ inbox placement rate at Litmus or Glock Apps

---

### TICKET 9 — Custom subdomain for tracking links

**Blocks:** Click-through rate visibility, deliverability
**Severity:** Medium

**What to build:**
- Set up a custom subdomain for Klaviyo's tracking links — e.g., `track.therunrec.com` or `email.therunrec.com`
- DNS CNAME record per Klaviyo instructions
- Configure in Klaviyo: Settings → Account → Custom Tracking Domain

**Definition of done:**
- Test email links go through the custom domain (not Klaviyo's default)
- Inspector tools show no domain mismatch warnings

---

### TICKET 10 — Custom sending domain (optional, recommended)

**Blocks:** Long-term sender reputation
**Severity:** Low (can launch without; recommended for franchise scale)

**What to build:**
- Set up `send.therunrec.com` or similar as a dedicated sending subdomain
- Configure Klaviyo to send from this subdomain instead of `therunrec.com` directly
- Warm up gradually over 4 weeks

**Definition of done:**
- Sending domain configured in Klaviyo
- Volume ramp plan: 100 sends day 1, double daily up to 4-5K, then full volume
- Bounce rate stays under 2%, complaint rate under 0.1%

---

## Skedda integration — overall architecture decision

The 4 tickets above (#1, #2, #3, #7) all depend on Skedda → Klaviyo data flow. Three implementation paths to choose from:

| Approach | Pros | Cons | Best for |
|----------|------|------|----------|
| **Direct webhook** (Skedda → Klaviyo Track API) | Lowest latency, no middleware | Requires Skedda webhooks to support custom payloads; less flexible | If Skedda supports rich webhook payloads natively |
| **Zapier/Make.com middleware** | Fast to set up, no code | Monthly cost ($30-100/mo), limited transformations, dependency on third-party uptime | Quick v1, prove value, then migrate |
| **Custom Vercel/Cloudflare Workers middleware** | Full control, cheap, fast | Requires engineering time to build + maintain | Long-term, multi-location franchise |

**Recommendation:** Zapier middleware for v1 (this week). Migrate to custom Workers if/when it becomes a bottleneck (>5K events/day or franchise expansion).

**Operator decision needed:** Sal + Rob, before Eyad starts integration work.

---

## Definition of "engineering ready for plug-and-play"

When ALL of the following are true, the admin can build flows without engineering blockers:

- [ ] Tickets 1-7 deployed and tested with at least 5 successful event fires each
- [ ] Ticket 8 (domain auth) — all green in Klaviyo dashboard
- [ ] Skedda integration approach decided and live (Zapier or custom)
- [ ] All 17+ custom profile properties created in Klaviyo (per pre-flight Section D)
- [ ] All 13 segments built in Klaviyo (per pre-flight Section F)
- [ ] All 3 base templates built in Klaviyo (per pre-flight Section H)
- [ ] Existing conflict flows paused (per inventory + per-flow guides)

---

## Ownership

- Engineering tickets: Eyad + Rob
- Domain auth + DNS: Rob (or Sal if he owns the domain registrar)
- Klaviyo property/segment/template setup: Admin (per pre-flight setup doc)
- Skedda config changes: depends on who manages Skedda — likely Sal or Izzy
- Sign-off: Sal

---

## Status tracking

Add a row at the top of this doc when work starts:

```
## Status — YYYY-MM-DD
- Tickets started: [list]
- Tickets completed: [list]
- Blockers: [list]
- ETA to engineering-ready: [date]
```
