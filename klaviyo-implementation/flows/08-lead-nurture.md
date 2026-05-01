# Flow 08 — Lead Nurture

**Build time estimate:** 4 hours (5 emails + 2 SMS, with flow-exclusion logic to configure)
**Difficulty:** Medium (mostly straightforward, but the Welcome ↔ Lead Nurture exclusion is fiddly)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- The `Newsletter` list (ID `SDFmeU`) exists. (It does — confirmed in inventory.)
- A segment called `Lead Nurture Eligible` (we will create this in step 1 below).
- Profile property `lead_use_case` exists on Klaviyo profiles. (Klaviyo will create it automatically the first time it's set; we will trigger it via Touch 02 link clicks.)
- A 3-minute facility walkthrough video uploaded somewhere we can link to (YouTube unlisted is fine). Get the URL.
- Booking URL (`runrec.co/book` or whatever Skedda link the team is using publicly).

If the video doesn't exist yet, you can launch with a photo carousel link instead — but document the swap.

---

## What this flow does (plain English)

Someone signed up for the Newsletter but hasn't booked a session in 30+ days. Welcome Series already had its shot. This flow gives them seven more nudges over 21 days — a tour, a self-segmenting choice, a personal text, a price anchor, a customer story, a real offer, and one last call. After Day 21 they exit. They either book or get released back to general nurture.

The flow's job: take a curious lead and either convert them to a first-time booker, or stop wasting sends on someone who isn't going to convert.

---

## Trigger — what fires this flow

- **Type:** Segment-triggered (when a profile enters the `Lead Nurture Eligible` segment).
- **Why segment-triggered, not list-triggered?** Because the trigger needs three conditions (signed up, never paid, not in active Welcome). A segment lets us combine all three; a list-add trigger can't.

### Step 1 — Create the segment first (do this BEFORE building the flow)

In Klaviyo: **Audience → Segments → Create Segment → "Create from scratch."**

- **Name:** `Lead Nurture Eligible`
- **Definition (read this slowly — three conditions ANDed together):**

  Condition 1 — **What someone has done (or not done):**
  - Click "What someone has done (or not done)"
  - Pick metric: `Successfully Paid`
  - Set to: **has done** zero (0) times **over all time**

  Condition 2 — **If someone is in or not in a list:**
  - Click "If someone is or is not in a list"
  - Set to: **is in** list `Newsletter` (ID `SDFmeU`)

  Condition 3 — **Properties about someone:**
  - Click "Properties about someone"
  - Pick: `Subscribed to Email Marketing` (or the Klaviyo equivalent for "joined a list more than X days ago")
  - Actually use: **What someone has done (or not done)** → metric `Subscribed to List` → list `Newsletter` → over date range → **more than 30 days ago**

  Condition 4 (the exclusion — this is the critical one) — **Has not done:**
  - Click "What someone has done (or not done)"
  - Pick metric: `Received Email`
  - Set: **has done** more than 0 times where **Flow ID equals `RZsSVX`** (the Welcome flow) **in the last 30 days**
  - Then flip the operator to: **has not done**

- Click **Create Segment**. Klaviyo will start building it. Could take a few minutes.

### Step 2 — Set the flow trigger

Once the flow is created (next section), set the trigger to:

- **Trigger type:** Segment
- **Segment:** `Lead Nurture Eligible` (the one you just made)
- **Direction:** When someone joins this segment

---

## Filters

These filters live INSIDE the flow (separate from the trigger segment, as a second safety net).

In Klaviyo, after dragging in the trigger, click **"Add Filter"** at the top of the flow:

- **Filter 1:** Profile is in segment `Lead Nurture Eligible` (yes — duplicate of trigger, as a safety check).
- **Filter 2:** Has not received email from Flow `RZsSVX` (Welcome) in last 30 days. (Same exclusion, second layer.)
- **Filter 3:** Email marketing consent = `subscribed`.

Yes, the Welcome exclusion is checked twice. That's intentional. If a profile is already in Welcome when this flow tries to fire, we want them OUT.

---

## Exit conditions

In Klaviyo, click **"Add Exit Condition"** at the top of the flow:

- **Exit if:** Has placed a `Successfully Paid` event (any time after entering the flow). This means: they booked. Stop sending.
- **Exit if:** Email marketing consent = `unsubscribed`. (Klaviyo handles this automatically but we'll be explicit.)

---

## Smart Sending: ON

**Why ON for this flow:** These are people who haven't bought yet — over-mailing them inside a 24-hour window kills the relationship. Smart Sending means Klaviyo won't send a flow email to a profile who already received any other Klaviyo email in the past 16 hours.

**How to set it:** Inside each email touch, on the right-side panel, look for **"Skip recently sent."** Toggle it ON.

## Quiet Hours: 7am–9pm America/Toronto

In each email and SMS touch, set send-time window:
- Earliest send: 9:00 AM
- Latest send: 7:00 PM
- Timezone: Recipient's local timezone (use the profile's `location.timezone` — Klaviyo defaults to this when "Use recipient timezone" is checked)

For SMS specifically (Touches 03 and 07), Klaviyo enforces SMS quiet hours by default. Don't override.

---

## Flow map (visual)

```
Trigger (joins Lead Nurture Eligible segment)
  ↓
Filter (welcome exclusion, second pass)
  ↓
Touch 01 (Day 1) → Email: Facility walkthrough
  ↓ Wait 2 days
Touch 02 (Day 3) → Email: 4 use cases (self-segment via clicks)
  ↓ Wait 2 days
Touch 03 (Day 5) → SMS: Conversation opener (Sal)
  ↓ Wait 2 days
Touch 04 (Day 7) → Email: Off-peak slots + price anchor
  ↓ Wait 3 days
Touch 05 (Day 10) → Email: Customer story
  ↓ Wait 4 days
Touch 06 (Day 14) → Email: First-session offer (reply-to-redeem)
  ↓ Wait 7 days
Touch 07 (Day 21) → SMS: Last call
  ↓
EXIT
```

---

## Touch-by-touch build instructions

### Touch 01 — Facility walkthrough

- **Channel:** Email
- **Send delay:** Day 1 — Wait `1 day` after segment join, send at 10:00 AM recipient local time
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `What walking into RunRec actually looks like.`
- **Preview text:** `3-minute video tour. Door code, court layout, where the bathroom is.`
- **Template:** Flow / Personal (the minimalist HTML base — see template note at bottom of this file)

#### Body content (HTML — paste exactly, swap `{{walkthrough_video_url}}` and `{{first_name}}`)

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Most people who don't book on day 1 don't book at all. Usually because they don't know what they're walking into — RunRec is unstaffed, so there's no front desk to greet you.</p>

<p>Here's what it actually looks like inside:</p>

<p>→ <a href="{{walkthrough_video_url}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">3-minute walkthrough video</a></p>

<p>Door access, court layout, where to put your shoes, where the bathroom is, where to plug in your phone for music.</p>

<p>After watching this you'll know more than 80% of first-timers do when they show up.</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you've got questions, the fastest way to get them answered is to reply to this email. We read every one.</p>
```

#### Klaviyo UI walkthrough for Touch 01

1. From the empty flow canvas, click **"+"** below the trigger.
2. Select **"Email"** from the action menu.
3. On the right side, name the message: `Flow 08 — T01 — Facility Walkthrough`.
4. Click **"Edit Email."**
5. Top form: fill in From name, From email, Reply-to, Subject, Preview text (above).
6. Click **"Edit Content"** → choose template **Flow / Personal** (or the template you've designated for personal-feel emails).
7. Drag a single Text block in. Switch the editor to "Source" / HTML mode and paste the HTML above.
8. Click **"Done."**
9. Back on the flow canvas, click the email's grey settings gear → set **Smart Sending: ON**.
10. Above the email, drag in a **"Time Delay"** — set it to **1 day** with send window **9 AM – 7 PM recipient timezone**.

---

### Touch 02 — 4 use cases (self-segment)

- **Channel:** Email
- **Send delay:** Wait `2 days` after Touch 01 → Day 3 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `4 ways people use this place.`
- **Preview text:** `Click yours. We tailor what we send.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>RunRec players fall into four camps. Click whichever sounds like you and we'll send the good stuff:</p>

<p>→ <strong>Solo training:</strong> I'm here to work on my game alone. <a href="{{root_url}}/lead?use_case=solo" style="color:#e85a3e;text-decoration:underline;font-weight:600;">[click]</a><br>
→ <strong>Crew player:</strong> I'm always here with my friends. <a href="{{root_url}}/lead?use_case=crew" style="color:#e85a3e;text-decoration:underline;font-weight:600;">[click]</a><br>
→ <strong>Party host:</strong> I book parties for kids or adults. <a href="{{root_url}}/lead?use_case=party" style="color:#e85a3e;text-decoration:underline;font-weight:600;">[click]</a><br>
→ <strong>League runner:</strong> I run a recurring league or training group. <a href="{{root_url}}/lead?use_case=league" style="color:#e85a3e;text-decoration:underline;font-weight:600;">[click]</a></p>

<p>Pick wrong and you'll get the wrong stuff for a while — no big deal.</p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Highest-revenue group is "League runner." Just an FYI.</p>
```

#### How the click-to-tag works

The four `[click]` links go to URLs like `runrec.co/lead?use_case=solo`. We need a tracking pixel or page handler that:
- Either redirects to the booking page after recording the param.
- Or just shows a "Got it — we'll tailor what we send" page.

**Klaviyo side: turn link clicks into a profile property update.** In each email link, click the link → "Add tracking" → use Klaviyo's "Custom property update on click" feature. Set:
- Property name: `lead_use_case`
- Property value: `solo` / `crew` / `party` / `league` (one per link)

If your Klaviyo plan doesn't include the click-to-property feature, fall back to: each link goes to a unique landing page, and your website fires a `set_property` event back to Klaviyo via the JS snippet.

#### Klaviyo UI walkthrough for Touch 02

1. After Touch 01, drag in **Time Delay → 2 days**.
2. Click **"+"** → **Email**.
3. Name: `Flow 08 — T02 — 4 Use Cases`.
4. Subject + preview text from above.
5. Body HTML pasted into a Text block.
6. **For each of the 4 links** in the email, click the link in the editor → "Tracking" → enable **"Update profile property on click"** → property `lead_use_case`, value matching the URL param.
7. Smart Sending: ON.
8. Save and continue.

---

### Touch 03 — Conversation opener (SMS)

- **Channel:** SMS
- **Send delay:** Wait `2 days` after Touch 02 → Day 5 total
- **Sender label / From:** `Sal – RunRec Co-Owner`
  - **In Klaviyo:** Settings → SMS → Sender Profiles → make sure a sender labeled "Sal – RunRec Co-Owner" exists. If not, create one and tie it to the RunRec long code (+1 519-800-9894).
- **Send time window:** 12:00 PM – 5:00 PM recipient local time

#### Body content (SMS — exact text, 160 chars matters)

```
Got any questions about getting started at RunRec? Just reply. -Izzy, RunRec · Stop=opt-out
```

#### Klaviyo UI walkthrough for Touch 03

1. After Touch 02, drag **Time Delay → 2 days**.
2. Click **"+"** → **SMS**.
3. Name: `Flow 08 — T03 — Conversation Opener SMS`.
4. **Sender Profile:** select "Sal – RunRec Co-Owner."
5. Paste body content into the message field.
6. Make sure the **STOP language** is included (`· Stop=opt-out`). Klaviyo may flag this as redundant since they auto-append; if it does, accept the duplication — CASL prefers visible opt-out on the first SMS in a flow.
7. Smart Sending: ON.
8. Quiet hours: enforced by Klaviyo SMS settings.
9. Save.

**Important:** This is the FIRST SMS the lead receives in this flow. Verify they have SMS marketing consent before this fires. Add filter inside the SMS step: **"Properties about someone → SMS Marketing → equals subscribed."** If false, skip this step (the flow continues without SMS — that's fine).

---

### Touch 04 — Off-peak slots + price anchor

- **Channel:** Email
- **Send delay:** Wait `2 days` after Touch 03 → Day 7 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `These slots are usually wide open.`
- **Preview text:** `4 specific times. 2 of them at $45/hr.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>The hardest part of starting at RunRec isn't the booking — it's deciding when.</p>

<p>Here's what's almost always available:</p>

<p>→ <strong>Tuesdays 8pm-10pm · Court B ($45/hr)</strong> — never crowded<br>
→ <strong>Wednesdays 10pm-12am · Court A ($89/hr)</strong> — peak quiet hours<br>
→ <strong>Thursdays 7am-9am · Court C ($45/hr)</strong> — early bird, full place to yourself<br>
→ <strong>Sundays 2pm-4pm · Court A ($89/hr)</strong> — surprisingly open most weeks</p>

<p>Pick one and book: <a href="{{booking_url}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">{{booking_url}}</a></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, First-timers who book mid-week have a 2× higher rebook rate than weekend-only bookers. Worth knowing.</p>
```

Replace `{{booking_url}}` in two places with the actual booking URL (e.g., `https://therunrec.com/book`).

#### Klaviyo UI walkthrough for Touch 04

Same pattern as T01. Drag delay 2 days, drag email, paste content, Smart Sending ON.

---

### Touch 05 — Customer story (epiphany bridge)

- **Channel:** Email
- **Send delay:** Wait `3 days` after Touch 04 → Day 10 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `How {{name}} went from never playing to 3x a week.`
- **Preview text:** `A real story from a real regular.`
- **Template:** Flow / Personal

**About the `{{name}}` in the subject:** This is NOT a Klaviyo merge tag. It's a placeholder for a real customer's first name (e.g., "Marcus"). Either hard-code it (e.g., subject becomes literally `How Marcus went from never playing to 3x a week.`) OR set up a custom event property — easiest: hard-code with one customer story to start, then build a rotation later.

For launch, pick one real customer (with permission) and use their actual name throughout. Replace every `{{name}}` below with the real name.

#### Body content (replace `{{name}}` with the actual customer's first name)

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Meet {{name}}. 6 months ago, {{name}} hadn't touched a basketball in 8 years.</p>

<p>Now {{name}} books Court B every Tuesday and Thursday at 9pm. Solo. With music. Just shooting around for an hour and clearing his head.</p>

<p>He told us: <em>"I didn't realize how much I missed it until I had a place where I didn't have to compete with five high school kids for a hoop."</em></p>

<p>That's the whole RunRec model. Private courts, no waiting, your own time.</p>

<p>{{name}}'s story isn't unique. Most of our regulars started exactly like you — just curious enough to book once.</p>

<p>Want to try? <a href="{{booking_url}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">{{booking_url}}</a></p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, {{name}}'s first booking was 11pm on a Wednesday. He couldn't sleep, decided to play. He's been a regular ever since.</p>
```

**Image to include (header above body):** Real photo of the customer on the court (with written permission). If no photo available, skip the image — pure copy is fine.

---

### Touch 06 — First-session offer (the close)

- **Channel:** Email
- **Send delay:** Wait `4 days` after Touch 05 → Day 14 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `About that first session.`
- **Preview text:** `Reply with your preferred day and time. 7-day expiry. First-timers only.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>You've been on our list for 14 days. Time to make this real.</p>

<p><strong>First-session offer:</strong></p>

<p>The math: a Court B session is normally $45. With this offer it's $33.75. <strong>Try one hour. If it's not for you, no harm done.</strong></p>

<p>Reply to this email with your preferred day and time and we'll redeem the offer for you.</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Most first-timers who book once come back within a couple of weeks. Just a pattern we see.</p>
```

#### Why no discount code?

This flow uses **reply-to-redeem** instead of a coupon code. The lead replies with their preferred time. The team applies the 25% discount manually at booking. No `FIRST25` code, no public link — keeps the offer clean and prevents code-leaking on Reddit. (See the buildsheet's "Mechanic Doctrine" section.)

The team needs an inbox-triage routine to actually catch and book these replies. **Operator action:** assign someone to monitor `contact@therunrec.com` for "reply with your preferred day" responses and book them in Skedda within 4 hours.

---

### Touch 07 — Last-day nudge (SMS)

- **Channel:** SMS
- **Send delay:** Wait `7 days` after Touch 06 → Day 21 total
- **Sender label:** `RunRec` (NOT "Sal" — by Day 21 the relationship is back to brand voice)
- **Send time window:** 9:00 AM – 12:00 PM recipient local time (morning send for "tonight's deadline" framing)

#### Body content (SMS)

```
Tonight's the last day to grab the offer — reply with your day/time. -RunRec
```

Note: The STOP language is NOT repeated here because Klaviyo auto-appends it to all SMS sends after the first one in a flow. If your account requires explicit STOP every time, append `· Stop=opt-out` to the end.

#### Klaviyo UI walkthrough for Touch 07

1. Drag Time Delay 7 days.
2. Click **"+"** → SMS.
3. Sender Profile: `RunRec` (default brand sender, NOT the Sal sender).
4. Paste body.
5. Send time window: 9 AM–12 PM recipient timezone.
6. Smart Sending: ON.
7. Filter: Profile is in segment `Lead Nurture Eligible` (still — they may have left it by Day 21, in which case skip).
8. Save → flow ends.

---

## Test plan

Before activating:

1. **Create a test profile** in Klaviyo: use your own email + a phone number you control. Set first_name to `TestLead`.
2. Add the test profile to the `Newsletter` list.
3. Wait for the segment to update (give it 30 mins, or manually trigger segment refresh in Klaviyo).
4. **Manually push the test profile through the flow** using Klaviyo's "Send a test email" or "Preview & test" feature on each touch.
5. Open every email on desktop AND mobile. Verify:
   - Subject line shows correctly
   - Preview text shows correctly
   - Body renders (no broken HTML)
   - Links are clickable and go to real URLs (not `#`)
   - Unsubscribe link works
6. For SMS touches: send a test SMS to your phone. Verify:
   - Sender label shows correctly
   - Body fits in one SMS segment (160 chars)
   - STOP works (text STOP back, verify you're unsubscribed)
7. **Test the click-to-tag on Touch 02:** click each of the 4 use-case links, verify the `lead_use_case` property updates correctly on your profile.
8. **Test the Welcome exclusion:** create a second test profile, add to Newsletter, immediately verify they ENTER Welcome flow (Welcome flow `RZsSVX`), then check the segment — they should NOT be in `Lead Nurture Eligible` until Welcome flow finishes + 30 days pass.

---

## Activation checklist

Before flipping the flow from Draft to Live:

- [ ] All 7 touches built and tested
- [ ] Smart Sending ON on every email + SMS
- [ ] Quiet hours set on every touch
- [ ] Trigger segment `Lead Nurture Eligible` created and populated
- [ ] Welcome exclusion verified (test profile in Welcome does NOT enter Lead Nurture)
- [ ] Walkthrough video URL is real and works (Touch 01)
- [ ] Booking URL is real and works (Touches 04, 05)
- [ ] Customer story name + photo (Touch 05) — real customer with written permission
- [ ] Inbox triage assigned for "reply to redeem" replies on Touch 06
- [ ] Lead Nurture Eligible segment population is reasonable (under 500 profiles for first launch — check before going live)
- [ ] Operator approval to activate

---

## Post-launch monitoring (first 7 days)

Check daily:

- **How many profiles entered the flow?** (should be 5-30/day for a healthy newsletter signup rate)
- **Touch 01 open rate.** Target 40%+ — these are warm leads. Below 25% = subject line problem or list quality issue.
- **Touch 02 click distribution.** What % clicked solo / crew / party / league? If one is wildly dominant (>70%), it's a self-selection bias and you can focus future content there.
- **Touch 03 SMS reply rate.** Target 5%+ replies. Reply rate is the leading indicator of conversion.
- **Touch 06 reply count.** This is the actual revenue moment. Track every reply, every booking outcome.
- **Unsubscribes per touch.** Above 0.5% on any single touch = problem touch. Pause and review.
- **Did Welcome exclusion hold?** Run a Klaviyo report: "Profiles in both Welcome flow `RZsSVX` AND Lead Nurture (this flow) in last 30 days." Should be ZERO. If not, the exclusion is broken — investigate.

---

## Common issues for THIS flow

**Issue: The segment is empty / not populating.**
- Check the segment definition. Klaviyo segment-builder is finicky — make sure you used "AND" between conditions, not "OR."
- The "more than 30 days ago" condition needs the segment to wait at least 30 days after a profile signs up. New accounts take time to populate.

**Issue: Profile is in Welcome AND Lead Nurture at the same time.**
- The exclusion filter is broken. Re-check the segment condition for "has not received from Flow `RZsSVX` in last 30 days."
- Check the in-flow filter (Filter 2 above) — both layers must be in place.

**Issue: SMS not sending on Touch 03.**
- Most likely cause: profile doesn't have SMS consent. The in-step filter ("SMS Marketing equals subscribed") will skip the touch. This is correct behavior. Flow continues to Touch 04 normally.
- Less likely: Sender Profile "Sal – RunRec Co-Owner" doesn't exist. Create it in Settings → SMS.

**Issue: Touch 06 replies are getting lost.**
- The team must monitor `contact@therunrec.com` actively. Set up Gmail labels/filters: any inbound with subject containing "About that first session" or sent in reply to Touch 06 → label as "Lead Nurture Reply" → assign to specific human.

**Issue: Use-case property not updating from clicks (Touch 02).**
- Check Klaviyo plan tier — "update profile property on click" requires certain plans. If unavailable, fallback: track clicks via Klaviyo's link tracking and manually segment via "Clicked Email" event filtered by URL.

---

## Template note (read once, applies to all touches)

All emails in this flow use the **Flow / Personal** template — minimalist, single column, RunRec logo at the top, address + unsubscribe at the bottom, no header image unless specified. If this template doesn't exist in the Klaviyo Templates library yet, build it before launching this flow. The buildsheet calls for three base templates: **Flow/Personal**, **Birthday Party**, and **VIP/Status**. This flow needs only **Flow/Personal**.

The footer on every email must contain:
- Business name: `RunRec`
- Physical address: `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8`
- Unsubscribe link
- "Manage preferences" link
- (CASL requirement)
