# Flow 09 — Corporate & League Nurture

**Build time estimate:** 6 hours (most complex flow in the system — multi-location sender doctrine + custom event integration + B2B SLA writing)
**Difficulty:** Hard (requires website form integration, custom event firing, sender-profile branching for franchise scaling)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- A B2B inquiry form on the website (`therunrec.com/corporate` or similar) that fires a custom event called `corporate_inquiry` to Klaviyo with these fields:
  - `email` (required)
  - `contact_name` (required)
  - `company_name` (optional)
  - `event_size` (number, optional)
  - `event_date` (date, optional)
- Profile properties created in Klaviyo (Settings → Profile properties → Add):
  - `location_manager` (text, e.g., "Izzy Dader")
  - `location_manager_email` (text, e.g., `izzy@therunrec.com`)
  - `location_phone` (text, e.g., `+1 519-800-9894`)
  - `location_name` (text, e.g., "Waterloo")
  - `corporate_inquiry_date` (date)
  - `corporate_status` (text — values: `active`, `parked`, `lost`, `won`)
- A calendar booking link (e.g., Calendly) for 15-min discovery calls.
- A photo/case study from a real corporate group (with written permission).

If the website form doesn't exist yet, this flow CANNOT launch — there is no trigger event. **Operator action:** coordinate with web dev to ship the form + Klaviyo event.

---

## What this flow does (plain English)

A company submits a B2B inquiry on the website ("we want to host our team off-site at your facility"). This flow runs them through 7 touches over 21 days: an immediate detailed reply with FAQs, a 5-minute "got your note" SMS, a case study from a similar company, three pricing tiers, a one-question SMS to re-open the conversation, a "should we close the file?" decision-forcing email, and a final "parking this" handoff that sets up a 90-day re-engagement window.

The flow's job: convert corporate inquiries into booked corporate events. B2B AOV is dramatically higher than retail bookings ($740+ per event), so each inquiry is worth pursuing for 21 days — but no longer.

---

## Trigger — what fires this flow

- **Type:** Metric (custom event)
- **Metric:** `corporate_inquiry`
- **Configuration in Klaviyo:**
  1. Open Flows → Create Flow → "Build your own."
  2. Set trigger: **Metric** → choose `corporate_inquiry`.
  3. If the metric doesn't exist yet, you need to fire one test event from the website first to register it in Klaviyo. Until that's done, the trigger dropdown won't show it.

**To create the event from the website (developer task):**

```javascript
klaviyo.identify({ email: form.email, first_name: form.contact_name });
klaviyo.track('corporate_inquiry', {
  contact_name: form.contact_name,
  company_name: form.company_name,
  event_size: form.event_size,
  event_date: form.event_date,
  inquiry_source: 'website_form'
});
```

After the first test fire, the metric appears in Klaviyo Metrics within ~5 minutes.

---

## Filters

In the flow trigger configuration, click **"Add Filter"**:

- **Filter 1:** Has not received this flow in last 6 months. (Klaviyo: "What someone has done — Received Email — from this flow — zero times in last 180 days.")
- **Filter 2:** Email marketing consent = `subscribed`.
- **Filter 3:** Email matches valid business pattern (optional — Klaviyo doesn't natively block free emails, but you can filter `email contains @gmail.com OR @yahoo.com OR @hotmail.com` and add a side path that just sends Touch 01 only, skipping the rest, since free-email senders are usually low-quality leads). For launch, skip this filter.

---

## Exit conditions

In flow settings, click **"Add Exit Condition"**:

- **Exit if:** A `corporate_event_booked` custom event fires (means: they booked — stop nurturing).
  - This requires the team to fire that event manually from Skedda when a corporate event is locked in. Until that automation exists, manually mark `corporate_status = won` on the profile and add a second exit condition: **profile property `corporate_status` equals `won`**.
- **Exit if:** Email marketing consent = `unsubscribed`.

---

## Smart Sending: OFF — and why

**Why OFF for THIS flow:** B2B prospects are worth the dedicated send. If they've received another email from us in the last 16 hours (e.g., a newsletter), we still want this flow to send — the inquiry-driven sequence is high priority. Smart Sending would skip these.

**How to set:** Inside each email touch, **DISABLE** "Skip recently sent."

## Quiet Hours: 8am–6pm America/Toronto, weekdays only

B2B emails get opened at work. Don't send Saturday at 11pm.

In each touch, set:
- Earliest send: 8:00 AM
- Latest send: 6:00 PM
- Days of week: Monday – Friday only
- Timezone: Recipient timezone

**Exception:** Touch 01 (Day 0 immediate) and Touch 02 (Day 0 +5 min) — these need to fire IMMEDIATELY regardless of time of day. Speed of first reply is the single biggest B2B conversion lever. Set Touch 01 and Touch 02 to "Send immediately" with no quiet hours filter.

---

## Flow map (visual)

```
Trigger: corporate_inquiry custom event
  ↓
Filters (welcome exclusion, consent check)
  ↓
Touch 01 (Day 0 immediate) → Email: FAQ + calendar link [from {{location_manager}}]
  ↓ Wait 5 minutes
Touch 02 (Day 0 +5 min) → SMS: "Got your note"
  ↓ Wait 2 days
Touch 03 (Day 2) → Email: Case study [from {{location_manager}}]
  ↓ Wait 3 days
Touch 04 (Day 5) → Email: 3 tiered packages [from {{location_manager}}]
  ↓ Wait 4 days
Touch 05 (Day 9) → SMS: "Quick Q on team size"
  ↓ Wait 5 days
Touch 06 (Day 14) → Email: "Should we close the file?" [from RunRec Team]
  ↓ Wait 7 days
Touch 07 (Day 21) → Email: Parking + handoff [from {{location_manager}}]
  ↓
EXIT
```

---

## CRITICAL: Multi-location sender doctrine

5 of 7 emails in this flow send "from" `{{location_manager}} <{{location_manager_email}}>`. There are two ways to wire this in Klaviyo. Read both, pick one, document the choice.

### Option A — Single fallback sender + merge tag in From line (FASTER, less franchise-correct)

- Klaviyo lets you put a merge tag in the From: name (e.g., `From name: {{ person.location_manager|default:"RunRec" }}`).
- BUT — Klaviyo does NOT let you put a merge tag in the actual From: address. The address is a fixed sender, fully authenticated against your DKIM/SPF.
- So you'd send from `contact@therunrec.com` BUT show "Izzy Dader" as the visible from-name. The actual reply-to could be the merge tag email.
- **Trade-off:** Easier to set up. Single-location only. Doesn't scale to franchises (because all franchises end up using the same `contact@therunrec.com` From: address).

**Option A setup:**
1. From name: `{{ person.location_manager|default:"RunRec" }}`
2. From email: `contact@therunrec.com` (fixed)
3. Reply-to: `{{ person.location_manager_email|default:"contact@therunrec.com" }}`

### Option B — Branched sender per location (CORRECT for franchise scaling)

- Each location has its own sender profile in Klaviyo (Settings → Account → Email → Senders → Add):
  - "Izzy Dader" `<izzy@therunrec.com>` — Waterloo
  - "Future Manager Name" `<manager@kitchener.therunrec.com>` — Kitchener (when launched)
  - etc.
- Each sender's domain must be DKIM/SPF authenticated.
- Inside the flow, BEFORE each `{{location_manager}}` email, drop in a **Conditional Split**:
  - If `location_name == "Waterloo"` → send Touch from "Izzy Dader" sender
  - If `location_name == "Kitchener"` → send Touch from "Future Manager" sender
  - Else → fallback to "RunRec" sender from `contact@therunrec.com`
- This means the flow has 5 conditional splits (one per `{{location_manager}}` email), each branching to a per-location sender variant.

**Trade-off:** Way more setup, way more email touches in the canvas (5 × 2 locations = 10 touches today, more as you scale). But: when Kitchener launches, you copy the flow, rename the sender on each branch, and you're done. Franchise-correct.

### Recommendation: **Build with Option A for launch (single Waterloo location, fast to ship), plan migration to Option B before franchise expansion.**

Document the choice. The buildsheet recommends Option B for franchise scaling — be honest with the operator about which one you picked.

---

## Touch-by-touch build instructions

### Touch 01 — Personal-feel reply + FAQ

- **Channel:** Email
- **Send delay:** Day 0 — IMMEDIATELY on trigger fire (no delay). No quiet hours.
- **From name:** `{{ person.location_manager|default:"RunRec Team" }}` (Option A) OR conditional per location (Option B)
- **From email:** Option A: `contact@therunrec.com` with reply-to `{{ person.location_manager_email|default:"contact@therunrec.com" }}` | Option B: per-location sender
- **Subject line:** `Got your inquiry, here's everything you need.`
- **Preview text:** `Top 3 FAQs answered upfront. Calendar link inside.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ event.contact_name|default:'there' }},</p>

<p>Got your team event inquiry — thanks for thinking of RunRec.</p>

<p>Quick answers to the top 3 questions we usually get:</p>

<p>→ <strong>How big can the group be?</strong> Up to 60 people across all 3 courts simultaneously.<br>
→ <strong>What's pricing?</strong> Recurring sessions: $89/hr Court A. Quarterly off-sites: $740 for 3 hrs + coordinator (most popular). Custom packages for 25+ people.<br>
→ <strong>What's included?</strong> Private facility, 3 courts, Bluetooth speaker access, parking, optional coordinator.</p>

<p>Want to chat? Pick a 15-min slot here: <a href="{{calendar_link}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">{{calendar_link}}</a></p>

<p>Or just reply with your event date and team size and I'll send a tailored proposal.</p>

<p style="font-style:italic;margin-top:20px;">Sal – RunRec Co-Owner</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Our most-booked corporate slot is Friday 4-7pm. Books up about 3 weeks ahead.</p>
```

**Note on the `Sal – RunRec Co-Owner` body signature vs `{{location_manager}}` From:** The buildsheet flags this inconsistency. For Waterloo launch, the body sig stays "Sal – RunRec Co-Owner" since Sal is the actual manager. For Kitchener (when it launches), the body would change to `{{location_manager}}`. Decision: hard-code "Sal" for now, swap to `{{location_manager}}` when location 2 ships.

Replace `{{calendar_link}}` with the actual calendar URL (e.g., `https://cal.com/runrec/discovery`).

#### Klaviyo UI walkthrough for Touch 01

1. From the empty flow canvas (after trigger), click **"+"** → **Email**.
2. Name: `Flow 09 — T01 — FAQ + Calendar`.
3. Click **"Edit Email."**
4. From name: paste merge tag exactly as shown above.
5. From email: per Option A or B you chose.
6. Reply-to: `{{ person.location_manager_email|default:"contact@therunrec.com" }}`
7. Subject + preview text.
8. Edit content → Flow / Personal template → paste body HTML.
9. Save.
10. **CRITICAL:** Set delay BEFORE this email to `0 seconds` — must fire immediately on trigger.
11. Smart Sending: OFF.
12. Quiet hours: OFF (override Klaviyo's defaults — go to message settings → "Send during quiet hours" = YES).

---

### Touch 02 — "Got your note" SMS

- **Channel:** SMS
- **Send delay:** Wait `5 minutes` after Touch 01 → Day 0 +5 min
- **Sender label:** `RunRec Team`
- **Send time:** No quiet hours — fires immediately

#### Body content

```
Got your note — full email coming with details. -RunRec Team · Reply STOP to opt out
```

#### Klaviyo UI walkthrough for Touch 02

1. Drag Time Delay → set to **5 minutes**.
2. Click **"+"** → **SMS**.
3. Name: `Flow 09 — T02 — Got Your Note`.
4. Sender Profile: `RunRec Team` (this needs to exist in Settings → SMS → Senders).
5. Body content from above.
6. Smart Sending: OFF.
7. Quiet hours: OFF.
8. Filter inside SMS step: `Properties → SMS Marketing == subscribed`. If false, skip (flow continues without SMS).

**SMS consent caveat:** Many B2B inquiry forms only collect email, not phone. If `phone_number` is empty on the profile, this SMS step is automatically skipped. That's fine — the flow continues without SMS.

---

### Touch 03 — Case study (similar org)

- **Channel:** Email
- **Send delay:** Wait `2 days` after Touch 02 → Day 2 total
- **From name:** `{{ person.location_manager|default:"RunRec Team" }}` (Option A) OR per-location (Option B)
- **From email:** Same as Touch 01
- **Subject line:** `How {{example_company}} runs their team night here.`
- **Preview text:** `28-person team. $6.35/person. Recurring Thursday.`
- **Template:** Flow / Personal

**About `{{example_company}}` in the subject:** Same caveat as the customer-name in Flow 08 T05. This is NOT a Klaviyo merge tag — it's a placeholder for a real example company. Hard-code one example company for launch (e.g., "How Acme Corp runs their team night here.") with their permission. Replace `{{example_company}}` with the actual name everywhere.

#### Body content

```html
<p>Hey {{ event.contact_name|default:'there' }},</p>

<p>Quick example of what a team night looks like — pulling from {{example_company}}'s recurring booking:</p>

<p><strong>Group:</strong> 28 people, marketing + product team<br>
<strong>Cadence:</strong> Every other Thursday, 5-7pm<br>
<strong>Format:</strong> 1 hour basketball pickup, 1 hour volleyball<br>
<strong>Cost:</strong> $178 total ($89/hr × 2 hrs, Court A)<br>
<strong>Per-person:</strong> $6.35</p>

<p>Their VP told us: <em>"It's the cheapest team activity we've ever run. People actually show up."</em></p>

<p>The setup is the same for any group your size. Same court, same equipment, no coordinator needed (you'd run it yourselves).</p>

<p>Want me to draft a similar plan for your team? Reply with your size and frequency.</p>

<p style="font-style:italic;margin-top:20px;">Sal – RunRec Co-Owner</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you're 25+ people, our quarterly off-site package ($740, 3 hrs + dedicated coordinator) bundles everything. Most companies start without and add later.</p>
```

**Image:** Real photo of a corporate event on Court A (with written permission). Add as header image.

#### Voice mix note for Touch 03

The body signature is `Sal – RunRec Co-Owner` (same as Touch 01). The From: header uses the location-manager merge tag. Operator should confirm this hybrid is OK — the alternative is to fully match (Sal in From + Sal in body) for Waterloo launch and swap both to `{{location_manager}}` for franchise launches. Document the call.

---

### Touch 04 — Tiered packages

- **Channel:** Email
- **Send delay:** Wait `3 days` after Touch 03 → Day 5 total
- **From name:** `{{ person.location_manager|default:"RunRec Team" }}`
- **From email:** Same as Touch 01
- **Subject line:** `3 ways to do team sports at RunRec.`
- **Preview text:** `Happy Hour · Quarterly Off-site · Recurring League.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ event.contact_name|default:'there' }},</p>

<p>Three ways companies use the space:</p>

<p><strong>1. HAPPY HOUR</strong> → $89/hr (1 court, 1 hour). Best for small groups (8-15 people) trying us out, low commitment.</p>

<p><strong>2. QUARTERLY OFF-SITE</strong> → $740 (3 hrs + coordinator). Best for big team events, project wraps, all-hands days.</p>

<p><strong>3. RECURRING LEAGUE</strong> → Custom rate based on size + cadence. Best for teams who want a regular Wednesday at 6pm slot, every week.</p>

<p>Most companies start with #1 (low commitment) and graduate to #3 once it's a thing.</p>

<p>Want us to put together a proposal for one of these?</p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Recurring leagues get priority booking — same time every week, locked in for the season. We only allocate 4 league slots per court, so they go fast.</p>
```

**Voice note:** Body signed "Izzy, RunRec" per the source HTML. The buildsheet flags this as `— Sal` per spec — there's an inconsistency between the dashboard HTML and the buildsheet. Decision needed from operator: which sig?

For launch, use **"Izzy, RunRec"** to match the dashboard (since the dashboard is what was approved with Sal). Document the call and resolve later.

---

### Touch 05 — "Quick Q" SMS

- **Channel:** SMS
- **Send delay:** Wait `4 days` after Touch 04 → Day 9 total
- **Sender label:** `Sal – RunRec Co-Owner`
- **Send time:** Standard quiet hours apply (8 AM – 6 PM weekdays)

#### Body content

```
Quick Q on your team event — how big's the group? -Izzy
```

**Note on sender vs sig:** Sender label is "Sal – RunRec Co-Owner," signed "-Izzy." This intentional mismatch is in the source. The buildsheet flags it ("currently sender 'Sal · RunRec' / signed '-Sal' per spec"). Resolve before launch — mismatch will confuse recipients.

For launch, use sender label `Sal – RunRec Co-Owner` and signature `-Sal` to match the buildsheet. Update the body to:

```
Quick Q on your team event — how big's the group? -Sal
```

#### Klaviyo UI walkthrough for Touch 05

1. Drag Time Delay → 4 days.
2. Click **"+"** → SMS.
3. Sender Profile: `Sal – RunRec Co-Owner`.
4. Body from corrected version above.
5. Smart Sending: OFF.
6. Filter: SMS Marketing subscribed.

---

### Touch 06 — "Should we close the file?"

- **Channel:** Email
- **Send delay:** Wait `5 days` after Touch 05 → Day 14 total
- **From name:** `RunRec Team` (NOT location_manager — this is the team voice for the decision-force email)
- **From email:** `contact@therunrec.com`
- **Reply-to:** `{{ person.location_manager_email|default:"contact@therunrec.com" }}`
- **Subject line:** `Should we close the file on this one?`
- **Preview text:** `Three checkboxes. Easier to say no than ignore.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ event.contact_name|default:'there' }},</p>

<p>Just trying to figure out if your team event is still on the table or if we should park it for now.</p>

<p>Three options — pick whichever fits:</p>

<p>[ ] Yes, still interested. Send me a proposal.<br>
[ ] Not yet — circle back in {{quarter}}.<br>
[ ] Not a fit. Close the file.</p>

<p>Easier to know now than to keep emailing if you've moved on. <strong>No hard feelings either way.</strong></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you say "circle back in Q3," we'll set a reminder and check in then. Promise it won't be more than that.</p>
```

**Note:** Body says "Izzy, RunRec" per dashboard but buildsheet says signed "RunRec Team" — same inconsistency as T04. Match the buildsheet: sign `— RunRec Team`, swap "Izzy, RunRec" out.

`{{quarter}}` is a placeholder — hard-code to current upcoming quarter (e.g., "Q3" if you're in May). Easier than building dynamic quarter logic for the first version.

---

### Touch 07 — Final touch + parking (the canonical `{{location_manager}}` deployment)

- **Channel:** Email
- **Send delay:** Wait `7 days` after Touch 06 → Day 21 total
- **From name:** `{{ person.location_manager|default:"RunRec Team" }}` (Option A) OR per-location (Option B) — this is THE most franchise-correct touch in the flow
- **From email:** `{{ person.location_manager_email|default:"contact@therunrec.com" }}` (Option A merge tag) OR per-location sender (Option B)
- **Subject line:** `Parking this — feel free to circle back anytime.`
- **Preview text:** `90-day re-engagement set. No is rarely no forever.`
- **Template:** Flow / Personal

#### Body content (note `{{location_manager}}` and `{{location_phone}}` merge tags!)

```html
<p>Hey {{ event.contact_name|default:'there' }},</p>

<p>Parking this for now. If your team event plans change, just reply — I'll be here.</p>

<p>Email works ({{ person.location_manager_email|default:"contact@therunrec.com" }}) or text: {{ person.location_phone|default:"+1 519-800-9894" }}.</p>

<p style="font-style:italic;margin-top:20px;">— {{ person.location_manager|default:"RunRec Team" }}</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Our quarterly review of new corporate inquiries goes out the first week of every quarter. If anything changes on your end, hit me up.</p>
```

**This is the canonical multi-location email.** Every merge tag resolves to the local market. When Kitchener launches:
- `{{location_manager}}` → "Future Manager Name"
- `{{location_manager_email}}` → `manager@kitchener.therunrec.com`
- `{{location_phone}}` → Kitchener phone number

The flow ships once, the location data ships per-profile, the email is correct everywhere.

#### Klaviyo UI walkthrough for Touch 07

1. Drag Time Delay → 7 days.
2. Click **"+"** → Email.
3. Name: `Flow 09 — T07 — Parking + Handoff`.
4. From name: paste merge tag.
5. From email: per Option A or B.
6. Reply-to: same merge tag email.
7. Subject + preview text.
8. Edit content → paste body HTML.
9. Smart Sending: OFF.
10. Quiet hours: 8 AM – 6 PM weekdays.
11. **Profile property write-back:** add an "Update Profile" action AFTER this email → set `corporate_status = parked` on the profile.

---

## After Touch 07 — segment hand-off

After Touch 07 fires, the profile should be added to a long-term `B2B Nurture - 90 Day` segment for quarterly check-in campaigns (not flows, just manual or scheduled campaigns). Build this segment separately:

**Segment definition:**
- `corporate_status` equals `parked` AND
- `corporate_inquiry_date` was more than 90 days ago AND
- Email marketing consent = `subscribed`

This becomes the audience for a Q1/Q2/Q3/Q4 "Hey, anything new?" campaign you send manually 4x per year.

---

## Test plan

Before activating:

1. **Fire a test `corporate_inquiry` event manually.** Use the Klaviyo Track API or have web dev fire one from a test form submission with your email.
2. Verify the trigger fires and the profile enters the flow.
3. **Test EVERY touch via Klaviyo's "Preview & Send Test."**
4. Verify Touch 01 fires within 1 minute of trigger (Day 0 immediate). If it takes more than 5 min, check Smart Sending isn't accidentally on.
5. Verify Touch 02 SMS fires 5 min after Touch 01 (set quiet hours OFF for both).
6. Open every email on desktop AND mobile.
7. Send Touch 02 SMS to your real phone. Verify sender shows `RunRec Team`.
8. Send Touch 05 SMS. Verify sender shows `Sal – RunRec Co-Owner` (different sender from T02 — important).
9. **Verify merge tags resolve correctly.** On the test profile, set `location_manager = "Test Manager"` and `location_manager_email = "test@therunrec.com"`. Touch 07 should show "Test Manager" everywhere.
10. **Test the exit condition:** manually set `corporate_status = won` on the test profile mid-flow. Verify the profile exits within 24 hours.
11. **Test the re-entry filter:** try to fire a second `corporate_inquiry` event on the same profile. Verify the flow does NOT re-trigger (filter: "not received this flow in last 6 months").

---

## Activation checklist

Before flipping to Live:

- [ ] All 7 touches built
- [ ] Website B2B form is live and firing `corporate_inquiry` events
- [ ] At least 1 successful test event recorded in Klaviyo metrics
- [ ] Profile properties created (`location_manager`, `location_manager_email`, `location_phone`, `location_name`, `corporate_inquiry_date`, `corporate_status`)
- [ ] Sender Profiles created in Klaviyo SMS settings: `RunRec Team` AND `Sal – RunRec Co-Owner`
- [ ] Sender doctrine choice documented (Option A or Option B)
- [ ] Voice/sig inconsistencies resolved (T03 body sig, T04 body sig, T05 sender vs sig, T06 body sig)
- [ ] Calendar link is real (Touch 01)
- [ ] Case study photo + permission obtained (Touch 03)
- [ ] Smart Sending OFF on all 7 touches
- [ ] Quiet hours OFF on Touch 01 + Touch 02 (immediate fire)
- [ ] Quiet hours 8 AM–6 PM weekdays on Touches 03–07
- [ ] Inbox triage assigned for replies (Touches 01, 03, 04, 06, 07 all explicitly request replies)
- [ ] `B2B Nurture - 90 Day` segment created for post-flow re-engagement
- [ ] Operator approval

---

## Post-launch monitoring (first 7 days)

- **How many `corporate_inquiry` events fired?** Should match form submissions on the website. If form fired but Klaviyo didn't track it, the JS event isn't wired right.
- **Touch 01 deliverability.** B2B emails to corporate inboxes face stricter spam filtering. Check delivery rate — should be 95%+. If lower, sender domain is not warmed up enough for B2B.
- **Touch 01 reply rate.** Target 10-15% replies. Below 5% = subject line problem or the FAQ isn't matching what they actually asked.
- **Touch 06 response rate.** Target 25%+ — the decision-force email gets responses where polite follow-ups don't. If below 10%, the email isn't reaching them (deliverability) or the format isn't working (try a plain text version).
- **Touch 07 fire count.** This is the "we couldn't close them" pile. If 80%+ of inquiries reach Touch 07, the funnel is broken — review what's happening in Touches 01–06.
- **`corporate_status` distribution.** After 21 days, what % of inquiries are `won` vs `parked` vs `lost`? Healthy: 15-25% won, 60-70% parked, 10-15% lost.

---

## Common issues for THIS flow

**Issue: Trigger doesn't fire.**
- The `corporate_inquiry` metric must exist in Klaviyo BEFORE you save the trigger. If the dropdown doesn't show it, fire one test event from the website (or via Klaviyo's API explorer) to register it.
- Double-check the metric name spelling — case-sensitive: `corporate_inquiry` not `Corporate Inquiry`.

**Issue: Touch 01 doesn't fire immediately — delays by hours.**
- Smart Sending is ON. Turn it OFF.
- OR: Quiet hours filter is on and the trigger fired during quiet hours. For T01 + T02, ALWAYS turn quiet hours off ("Send during quiet hours = YES").

**Issue: From name shows literally `{{ person.location_manager }}` in the inbox.**
- The merge tag syntax is wrong. Klaviyo uses `{{ person.PROPERTY_NAME }}` for profile properties, NOT `{{PROPERTY_NAME}}`. Re-check syntax.
- Or: the property doesn't exist on the profile. The `default:"RunRec Team"` fallback should catch this — verify it's in the merge tag.

**Issue: SMS not going out.**
- Profile has no `phone_number`. Most B2B forms only collect email. Fine — flow continues without SMS.
- Sender Profile not configured. Settings → SMS → Senders → verify both `RunRec Team` and `Sal – RunRec Co-Owner` exist.

**Issue: Profile receives this flow twice in a month.**
- 6-month re-entry filter is broken. Re-check filter setup at trigger.

**Issue: Voice inconsistencies in body sigs.**
- This is documented above — the source HTML and buildsheet have small mismatches. Resolve with operator before launch. Better to ship with Sal-in-body (matches buildsheet) and `{{location_manager}}`-in-From (matches franchise plan).

---

## Template note

All emails use **Flow / Personal** template. Footer required on every email:
- Business name: `RunRec`
- Address: `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8`
- Unsubscribe link
- Manage preferences link
