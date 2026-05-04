# 02 — The Build Template

This file is the **structure that every per-flow guide follows**. Read it once, all the way through. Once you understand the pattern, every per-flow guide in the `flows/` folder will feel familiar — same sections, same order, every time.

**You will not build a flow from this file directly.** This file is the blueprint. The actual flows are at `flows/02-booking-confirm.md`, `flows/03-post-session.md`, etc. Each one fills in this template with the specific details for that flow.

**Why a shared template:** The 13 flows are different in topic, but the *building process* is identical. Once you know the rhythm — open the guide, set the trigger, set the filters, build each touch in order, run the test plan, complete the activation checklist — you'll move through each flow in 30-120 minutes.

---

## The 12 sections every per-flow guide contains

| # | Section | What it covers |
|---|---------|----------------|
| 1 | Flow purpose | One paragraph. Why this flow exists, what business outcome it drives. |
| 2 | Trigger | What event or condition starts this flow for a profile. |
| 3 | Filters | Who is allowed to enter the flow (and who is excluded). |
| 4 | Exit conditions | When a profile is removed from the flow before completing it. |
| 5 | Smart Sending setting | On or off, with reason. |
| 6 | Quiet Hours setting | The send-time window. |
| 7 | Flow steps map | Visual: trigger → touch 1 → wait → touch 2 → split → etc. |
| 8 | Email touches | One sub-section per email: timing, sender, subject, preview, body, template, conditional logic. |
| 9 | SMS touches | One sub-section per SMS: timing, sender, body, opt-out language. |
| 10 | Test plan | Step-by-step pre-launch verification. |
| 11 | Activation checklist | Final pre-launch gate. |
| 12 | Post-launch monitoring | What to watch for the first 7 days. |

Each section below explains what to expect and what to do.

---

## Section 1 — Flow purpose

**What you'll see in the per-flow guide:** One short paragraph. Plain English: who is this flow for, what should happen as a result.

**What to do:** Read it. If you don't understand the purpose, **stop and ask Sal**. Building a flow you don't understand produces sends you don't understand, and you can't validate them.

Example (Flow 02 Booking Confirm): *"This flow goes to a customer the moment they finish booking a court. The job is to confirm the booking, give them everything they need to show up confidently (door access, court letter, time), and reduce no-show risk. It's transactional, not promotional."*

---

## Section 2 — Trigger

**What you'll see:** The exact thing that starts this flow. One of:

- **Metric trigger:** A custom event firing (e.g., `booking_created`). The most common.
- **List trigger:** A profile being added to a specific list (e.g., joining the Newsletter list triggers Welcome).
- **Segment trigger:** A profile entering a segment (e.g., entering "VIP / Power User" triggers Flow 07).
- **Date-based trigger:** A profile property matching today's date (e.g., `dob` matches today triggers the birthday flow).

**What to do in Klaviyo:**

1. **Flows → Create Flow → Build your own**
2. Name the flow per the per-flow guide (e.g., `Flow 02 - Booking Confirm + Reminders`)
3. Klaviyo will ask you to set the **Trigger**. Click **Add a trigger**
4. Choose the trigger type from the per-flow guide:
   - For a metric: search for the metric name (e.g., `booking_created`). If not found, the engineering work for Section G of pre-flight isn't done yet — stop.
   - For a list: choose **List** as trigger type, select the list
   - For a segment: choose **Segment**, select the segment
   - For date-based: choose **Date property**, select the property (e.g., `dob`)
5. Save trigger

**Verify:** Klaviyo shows the trigger at the top of the flow canvas. The wording should match the per-flow guide exactly.

---

## Section 3 — Filters

**What they are:** Filters decide who gets into the flow once the trigger fires. Even if the trigger fires for a profile, a filter can block them from continuing.

**What you'll see in the guide:** A list of conditions. Examples:
- "Profile is on the Newsletter list" (only newsletter subscribers continue)
- "Profile is NOT in the Sunset segment" (don't send to people about to be cleaned out)
- "Profile has NOT received a similar email in the last 7 days" (anti-overlap)
- "Profile has email marketing consent" (CASL/CAN-SPAM compliance)

**What to do in Klaviyo:**

1. In the flow canvas, find the trigger box at the top
2. Click **Add filter** (just below the trigger)
3. For each filter in the per-flow guide:
   - Choose the filter type (profile property / metric / list membership)
   - Configure the condition exactly as written
4. Save

**Two important rules:**

- **All filters are AND logic** (all conditions must be true to continue). If a guide says "X OR Y," it'll usually be expressed as a more complex single filter.
- **Filters are evaluated only at trigger time.** If a profile passes the filter, then later changes (e.g., gets added to the Sunset segment), they continue through the flow. To stop them, use **exit conditions** (Section 4).

---

## Section 4 — Exit conditions

**What they are:** Rules that pull a profile out of the flow before they finish. Different from filters — exits run continuously after a profile enters.

**What you'll see in the guide:** A list. Examples:
- "Profile makes a purchase" — exit Welcome flow when they convert (no more nurture needed)
- "Profile unsubscribes" — exit immediately
- "Profile enters another flow with same audience" — exit to avoid double-mailing
- "Profile completes the desired action" — e.g., Abandoned Booking flow exits when the booking completes

**What to do in Klaviyo:**

1. In flow canvas, find **Flow Filters** at the top → click **Add an exit criterion**
2. For each exit condition in the guide:
   - Choose the event or condition
   - Configure exactly as written
3. Save

**Why this matters:** Without exits, a customer who buys mid-flow keeps getting "complete your booking" emails. Embarrassing and complaint-worthy.

---

## Section 5 — Smart Sending setting

**What it is:** Klaviyo's anti-spam-yourself feature. Default: prevents the same person getting two of the same message within 16 hours (email) or 12 hours (SMS).

**What you'll see in the guide:** "Smart Sending: ON" (almost always) or "Smart Sending: OFF" (rare — only for transactional flows where every send must go through).

**What to do in Klaviyo:**

1. In the flow canvas, click each individual email or SMS step
2. In the right panel, find **Smart Sending**
3. Toggle ON or OFF per the per-flow guide
4. Save

**Critical exception — Booking Confirm (Flow 02) and similar transactional flows:** Smart Sending is **OFF**. A booking confirmation must always be delivered, even if the same customer booked an hour ago.

---

## Section 6 — Quiet Hours setting

**What it is:** Klaviyo holds sends until they fall inside the quiet-hours window per recipient timezone.

**What you'll see in the guide:** Almost always **9 PM - 9 AM recipient local time**. Rarely overridden.

**What to do in Klaviyo:**

1. Quiet hours are usually inherited from the global account setting (you set this in Pre-Flight Section I.2)
2. To override on a single flow step, click the step → right panel → **Send Time** → toggle **Use Quiet Hours**
3. Almost always: leave inherited

**Critical exception — SMS:** Quiet hours are mandatory by law in many jurisdictions. Never disable on an SMS step. Email can sometimes be sent during quiet hours for transactional content (booking confirms), but generally don't.

---

## Section 7 — Flow steps map

**What you'll see:** A visual map of the entire flow, from trigger through every touch. Looks like:

```
[TRIGGER: booking_created]
  ↓
[FILTER: profile has email consent AND not in Sunset]
  ↓
[T01 EMAIL: Booking confirmation] (delay: immediate)
  ↓
[WAIT: until day-of-booking, 4 hours before start]
  ↓
[T02 SMS: Reminder + arrival info] (delay: 4hrs before booking_start_time)
  ↓
[T03 EMAIL: Last-minute checklist] (delay: 1hr before booking_start_time)
  ↓
[CONDITIONAL SPLIT: Did booking_completed event fire?]
  ├─ YES → [EXIT — go to Flow 03 Post-Session]
  └─ NO  → [T04 EMAIL: "Did everything go OK?"] (delay: 24hrs after expected booking end)
```

**What to do:** Read the map carefully BEFORE you start clicking. Make sure you understand the shape of the flow. The map matches the order you'll build the touches in Klaviyo.

In Klaviyo's flow canvas, you'll drag-drop these elements in:
- **Email** — for an email touch
- **SMS** — for an SMS touch
- **Time delay** — for a wait
- **Conditional split** — for a branch
- **Update profile property** — for setting a value
- **Exit** — for ending the flow

Each is dragged from the right-hand panel into the canvas at the position the map shows.

---

## Section 8 — Email touches

**This is the biggest section in any per-flow guide.** One sub-section per email. Every sub-section has the same fields.

For each email, the per-flow guide gives you:

| Field | What it means | What to do |
|-------|---------------|------------|
| **Touch number** | T01, T02, T03... | Drag an Email block into the canvas at the correct position per the map |
| **Send delay** | When this email fires (e.g., "immediate after trigger" or "Day 3, 10 AM recipient time") | Configure the time-delay block before this email |
| **From name** | Sender name shown to recipient (e.g., `RunRec` or `Sal at RunRec`) | In email block → right panel → **From label** |
| **From email** | Sender email (e.g., `contact@therunrec.com`) | In email block → right panel → **From email** — choose from sender profiles created in Pre-Flight Section C |
| **Reply-to** | Where replies go | In email block → right panel → **Reply-to** |
| **Subject line** | The exact subject (already written in the dashboard) | Copy from dashboard URL → paste into **Subject line** field |
| **Preview text** | The exact preview text | Copy from dashboard URL → paste into **Preview text** field |
| **Body** | Reference to the dashboard URL where the body lives | Open the URL, copy the body text into the email content area; if using a template, plug into the appropriate text block |
| **Template** | Which of the 3 templates from Pre-Flight Section H to use | In the email block → choose **Template** → select the named template |
| **Conditional logic** | Whether this touch is gated on a previous condition (e.g., "only if T01 was opened") | Configure conditional split before this touch if needed |

### Where the body copy lives

For every email, the per-flow guide will give you a URL like:

> Open: https://saldader.github.io/runrec-email-pipeline/flow-01-welcome-fullbuild.html#t01

This points to the exact email on the deployed dashboard. **The dashboard is the canonical source for body copy.** Copy and paste from the dashboard into Klaviyo. Do not retype — typos compound.

### How to set a Send Delay

For each email:

1. Drag a **Time delay** block into the canvas at the correct position
2. Click the time delay → right panel → set:
   - For relative delays (e.g., "1 hour after previous"): use **Wait for a specific amount of time** → enter the duration
   - For specific times of day (e.g., "send at 10 AM recipient time"): use **Wait until a specific time** → enter time
   - For date-property delays (e.g., "7 days before profile.dob"): use **Wait until a date property** → enter the property and offset
3. Save

### How to set a Conditional Split

When the per-flow guide shows a branch:

1. Drag a **Conditional split** block into the canvas
2. Click it → right panel → configure the condition (e.g., "What someone has done: Opened Email — at least once — since the start of this flow")
3. Two paths appear in the canvas: **Yes** and **No**
4. Build the touches that go down each path

---

## Section 9 — SMS touches

Similar to email but with fewer fields and stricter rules.

For each SMS, the per-flow guide gives you:

| Field | What to do |
|-------|------------|
| **Touch number** | T0X — drag an SMS block into the canvas |
| **Send delay** | Configure the time-delay block before this SMS |
| **From / Sender label** | In SMS block → right panel → set sender label (e.g., `RunRec` or `RunRec Team`) |
| **Body** | Copy from dashboard URL — paste into the SMS body field |
| **Quiet hours respect** | Always ON for marketing SMS. (For purely transactional, sometimes off — guide will specify.) |

### SMS body rules — non-negotiable

1. **Maximum 160 characters** for a single SMS segment. Klaviyo will warn if you exceed; longer messages are split into multiple billed segments.
2. **The first SMS in any flow must include opt-out language**: "Reply STOP to opt out" or compact form ("Stop=opt-out"). Required by CASL (Canada) and TCPA (US).
3. **No links shorter than 8 characters** — short URLs (e.g., `bit.ly/x`) trigger spam filters. Use full URLs or RunRec branded short link `runrec.co/...`
4. **Sender label is the visible "from" name** in the customer's text app. Standard: `RunRec`. Sometimes `RunRec Team` per Voice Doctrine.

---

## Section 10 — Test plan

Every flow has a test plan at the end of the per-flow guide. **Do not skip it.** A failed test in your inbox is far cheaper than a failed send to 1,000 customers.

The standard test plan structure:

### Test 1 — Send Test from each touch

For every email and SMS in the flow:

1. Click the touch in the canvas
2. Top-right of the touch's content panel: **Send Test**
3. Send to your own email/phone
4. Confirm it arrives within 5 minutes
5. Verify:
   - Subject and preview text render correctly
   - Logo appears
   - Body copy matches the dashboard
   - All links work (click each one, confirm it goes where expected)
   - Footer has unsubscribe link, physical address, business name
   - No `{{ merge_tag }}` text leaking through (means a property isn't populated)
   - Mobile preview looks correct

### Test 2 — Trigger the flow with a test profile

1. Create a test profile in Klaviyo with your own email and a test marker (e.g., `email = klaviyo-test+flowname@therunrec.com`)
2. Manually fire the trigger for that profile:
   - For metric triggers: send a test event via Klaviyo's API or use the test trigger button in the flow canvas
   - For list triggers: add the test profile to the trigger list
   - For segment triggers: add the test profile to the segment (or set the property that puts them in the segment)
3. Watch the flow over the next 24-48 hours (depending on flow length). Receive each touch as it fires. Confirm timing.

### Test 3 — Filter exclusion test

1. Create a second test profile that should be **excluded** by one of the filters (e.g., already in Sunset segment)
2. Trigger the flow
3. Confirm the profile does NOT enter the flow (check **Flow Analytics → Recipients** — they should not appear)

### Test 4 — Exit condition test (if applicable)

1. Create a third test profile
2. Trigger the flow
3. After T01 arrives, manually fire the exit condition (e.g., make a purchase event)
4. Confirm subsequent touches do NOT fire for that profile

### Test 5 — Quiet hours and smart sending verification

For SMS flows:

1. Manually fire a trigger at 11 PM your time
2. Verify the SMS does NOT send immediately
3. It should queue for delivery at 9 AM the following day
4. Wait and confirm

For Smart Sending (any flow with multiple touches in a short window):

1. Trigger the flow with two events back-to-back for the same profile
2. Confirm the customer receives only the appropriate number of messages (Smart Sending blocks duplicates)

---

## Section 11 — Activation checklist

Before any flow goes from **Draft** to **Live**, every box must be green.

Standard checklist (each per-flow guide will add flow-specific items):

- [ ] All Pre-Flight Setup items (file 01) complete
- [ ] Trigger configured exactly per the guide
- [ ] All filters configured
- [ ] All exit conditions configured
- [ ] Smart Sending set per the guide
- [ ] Quiet Hours set per the guide
- [ ] Every email built — subject, preview, body, template all match the dashboard
- [ ] Every SMS built — body matches dashboard, opt-out language present in first SMS
- [ ] Send Test for every touch ran successfully (Test 1 above)
- [ ] Trigger test profile completed full flow (Test 2)
- [ ] Filter exclusion test passed (Test 3)
- [ ] Exit condition test passed (Test 4) — if applicable
- [ ] Quiet hours / Smart Sending test passed (Test 5)
- [ ] No `{{ merge_tag }}` placeholders visible in any test send
- [ ] Sender profile is correct (`contact@therunrec.com` or `{{location_manager}}` per Voice Doctrine)
- [ ] Footer compliant: unsubscribe link, physical address, business name
- [ ] Pre-send safety checklist (CMO master) reviewed
- [ ] Sal approved activation (required at SHADOW trust level — current state)

When all green, in the flow canvas: change status from **Draft** to **Live** (top right). Klaviyo will ask to confirm. Confirm only when every box above is green.

---

## Section 12 — Post-launch monitoring

For the first 7 days after a flow goes live, watch closely.

### Day 1
- Open the flow in Klaviyo. Click **Analytics** tab.
- Check **Recipients** — confirm profiles are entering the flow as expected (not 0, not way more than expected)
- Spot-check a few profiles in the flow → verify their journey looks correct

### Day 2
- Check open rates on T01. Should be high (50%+ for transactional flows like Booking, 35-45% for behavioral flows)
- If open rate is below 25%: stop the flow, investigate (sender reputation, deliverability, segment definition)

### Day 3-4
- Check click-through rates on touches with CTAs
- Check unsubscribe rate. Should be under 0.5% per touch. If above, audience-relevance issue — investigate.

### Day 7
- Pull full flow report: opens, clicks, conversions (if applicable), unsubscribes, spam complaints
- Compare against KPI targets (file 04 or buildsheet)
- Note any patterns
- Save report to vault: `~/vault/marketing/reviews/flow-XX-week-1-YYYY-MM-DD.md`

### If you see RED FLAGS at any point:
- **Spam complaints above 0.05%** → pause flow immediately, escalate to Sal
- **Unsubscribe rate above 1%** → pause and investigate
- **Open rate collapses (below 15%)** → likely deliverability issue, escalate to Sal
- **Customer complaint to support** ("I got the same email 4 times" / "Why am I getting this") → pause flow, investigate, fix before reactivating

---

## Example walk-through (one paragraph)

To make this template concrete, here's how it plays out for a fictional flow called "Welcome to Pickleball Night."

*The per-flow guide says: Purpose is to onboard signups to the Pickleball Night event series. Trigger is "added to list `Pickleball Night Signups`." Filter: profile has email consent. Exit condition: profile makes a Pickleball booking. Smart Sending ON, Quiet Hours 9-9. Flow shape is 3 emails over 7 days: T01 immediate "Welcome to Pickleball Nights," T02 day 3 "What to expect your first night," T03 day 7 "Lock in your first session." All 3 use the `RR - Flow Personal v1` template. The from-name is "RunRec," from-email is `contact@therunrec.com`, reply-to same. Each email's body lives at `https://saldader.github.io/runrec-email-pipeline/pickleball-welcome.html#t01` etc. — copy from the dashboard, paste into Klaviyo. Run Test 1 (send test of each), Test 2 (test profile through the full flow), Test 3 (try a profile without email consent — should be excluded), Test 4 (trigger then book — confirm exit). Activation checklist: all the standards plus Sal's approval. Post-launch: watch open rate on T01 (should hit 45%+), unsubscribe rate (under 0.5%), and the conversion rate to first booking.*

That's the rhythm. Same beats, every flow.

---

## You're ready

Once you've read this file, open the first per-flow guide in your build order:

**`flows/02-booking-confirm.md`**

Follow it section by section. Use this template file as the reference for what each section means. The first flow takes the longest (you're learning); the second is faster; by the fourth flow, the rhythm is automatic.

Good luck. Move slowly. Send tests.
