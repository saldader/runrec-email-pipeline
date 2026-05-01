# Flow 12 — Abandoned Booking

**Build time estimate:** 4 hours (3 touches, but the +48h conditional webhook check is the critical complexity)
**Difficulty:** Hard (requires Vercel webhook integration AND a real-time availability check at Touch 03)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- **`booking_abandoned` custom event firing into Klaviyo.** This is the single biggest blocker. The Vercel checkout layer needs to fire this event when a user reaches checkout but doesn't complete payment within 30 minutes. Required event payload:
  - `email` (required)
  - `sport` (e.g., "Basketball")
  - `court` / `court_letter` (e.g., "A")
  - `time_slot` / `date` + `time` (e.g., "2026-05-15", "19:00")
  - `total_value` / `total` (numeric, currency)
  - `resume_url` (URL to resume the booking — typically Skedda deep link or Vercel checkout resume)
  - `cart_token` (unique ID for this abandoned cart, used by webhook for availability check)
- **Availability webhook endpoint.** Touch 03 only fires if the slot is genuinely still open. This requires:
  - A webhook endpoint (Vercel function or similar) that takes a `cart_token` and returns `{ available: true | false }`
  - Endpoint URL accessible by Klaviyo's Webhook action (e.g., `https://api.therunrec.com/v1/availability/check`)
  - **Operator action:** Eyad/Rob need to build both pieces. Per the inventory doc, this is flagged as their action item.
- **PAUSE the existing Browse Abandoned flows BEFORE activating this.** Per the inventory: flows `RncQfn` (4 emails) + `WajxZ4` (3 SMS) are LIVE on `Viewed Product` trigger. They will collide with this flow. Pause them in Klaviyo BEFORE Flow 12 goes live.

---

## What this flow does (plain English)

A customer reaches the Vercel checkout page to book a court but doesn't complete payment within 30 minutes. This flow runs them through 3 touches over 72 hours: a "still want this slot?" email at +1 hour (peak intent), a reassuring "your slot is still open + here's what others booked" email at +24 hours, and a real-scarcity SMS at +48 hours that ONLY fires if the specific slot is genuinely still available.

The flow's job: recover 15-25% of abandoned bookings. Per the buildsheet, abandoned booking is the highest-RPR flow in any system ($3.65 average per recipient). This flow has been sitting in draft for 3 years — silent revenue leak.

---

## Trigger — what fires this flow

- **Type:** Metric (custom event)
- **Metric:** `booking_abandoned`
- **Configuration:** Triggers when this metric event is received for a profile

### How to set the trigger

In Klaviyo: **Flows → Create Flow → Build your own → Trigger: Metric → `booking_abandoned`.**

If the metric doesn't exist yet, you need to fire one test event first to register it. Use Klaviyo's Track API or have dev fire from a test cart abandonment.

---

## Filters

In flow trigger configuration:

- **Filter 1:** Has not done `Successfully Paid` for the same `cart_token` in the last 24 hours. (Catches edge case: customer abandoned, then completed booking via direct link — don't email them about a booking they actually made.)
- **Filter 2:** Has not received this flow in last 7 days. (Prevents spam if customer abandons multiple carts in a week.)
- **Filter 3:** Email marketing consent = `subscribed`.
- **Filter 4:** Has valid email (Klaviyo handles this automatically).

---

## Exit conditions

- **Exit if:** `Successfully Paid` event with matching `cart_token` fires (booking completed — stop sending).
- **Exit if:** Email marketing consent = `unsubscribed`.

The cart-token match is critical. Without it, exit-on-paid fires for ANY paid booking, which would break the flow if customer abandons cart A but completes cart B in parallel.

If your Klaviyo plan can't filter exit on event property (`cart_token` matching), fall back to: exit on any `Successfully Paid` event in last 24 hours. Acceptable trade-off; means same person abandoning 2 different carts wouldn't get the second flow if they completed the first.

---

## Smart Sending: OFF — and why

**Why OFF:** Cart abandonment is the single most time-sensitive flow in marketing. Recovery rate drops dramatically after the first hour. Smart Sending could suppress Touch 01 if the customer received any other Klaviyo email recently — unacceptable for this flow.

## Quiet Hours: OFF for Touches 01 and 02; ON for Touch 03

**Touch 01 (+1 hour) and Touch 02 (+24 hours):** Send IMMEDIATELY regardless of time. The 1-hour window is when intent is highest. If customer abandons at 11pm, Touch 01 fires at 12am. That's correct.

**Touch 03 (+48 hours):** SMS — ALWAYS respect quiet hours. Klaviyo enforces SMS quiet hours by default. Don't override.

---

## Flow map (visual)

```
Trigger: booking_abandoned event fires
  ↓
Filters
  ↓
Wait 1 hour
  ↓
Touch 01 (+1 hour) → Email: Still want this slot?
  ↓ Wait 23 hours
Touch 02 (+24 hours) → Email: Your slot's still open
  ↓ Wait 24 hours
WEBHOOK CHECK: Is slot still available?
  ├─ YES → Touch 03 (+48 hours) → SMS: Real scarcity
  └─ NO → SKIP Touch 03 → EXIT
  ↓
EXIT
```

---

## Touch-by-touch build instructions

### Touch 01 — Still want this slot?

- **Channel:** Email
- **Send delay:** Wait `1 hour` after trigger event
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `Still want this slot?`
- **Preview text:** `{{ event.sport }} · Court {{ event.court_letter }} · {{ event.date }} {{ event.time }}`

(Note: Klaviyo's preview text supports merge tags from event data. If your Klaviyo plan doesn't support event-merge-tags in preview text, hard-code preview text to a generic phrase like "Your court slot's waiting.")

- **Template:** Flow / Personal

#### Body content (uses event payload merge tags)

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Looks like you started a booking but didn't finish:</p>

<p>→ <strong>Sport:</strong> {{ event.sport|default:"Basketball" }}<br>
→ <strong>Court:</strong> {{ event.court_letter|default:"A" }}<br>
→ <strong>Date:</strong> {{ event.date|default:"" }}<br>
→ <strong>Time:</strong> {{ event.time|default:"" }}<br>
→ <strong>Total:</strong> ${{ event.total|default:"45" }}</p>

<p>The slot's still yours if you want it: <a href="{{ event.resume_url }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Resume booking</a></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If something didn't work in checkout, just reply. We'll fix it.</p>
```

#### Klaviyo UI walkthrough for Touch 01

1. From the empty flow canvas (after trigger), drag in **Time Delay → 1 hour**.
2. Click **"+"** → **Email**.
3. Name: `Flow 12 — T01 — Still Want This Slot`.
4. From name, email, reply-to as above.
5. Subject + preview text (use event merge tags if your plan supports it).
6. Edit content → Flow / Personal template.
7. Drag a Text block, switch to HTML mode, paste body content.
8. **CRITICAL:** verify event merge tags resolve. Click "Preview" → choose a real `booking_abandoned` event from your test fires. Confirm `{{ event.sport }}` shows the actual sport, not blank.
9. Smart Sending: **OFF**.
10. Quiet hours: **OFF** (override Klaviyo's defaults).
11. Save.

---

### Touch 02 — Your slot's still open

- **Channel:** Email
- **Send delay:** Wait `23 hours` after Touch 01 → +24 hours total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Your slot's still open.`
- **Preview text:** `Plus what others booked this week.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Quick update — your <strong>{{ event.sport|default:"Basketball" }}</strong> court at <strong>{{ event.date|default:"" }} {{ event.time|default:"" }}</strong> is still available.</p>

<p>What other people booked this week (similar to what you almost grabbed):</p>

<p>→ Pickleball, Court A, Saturday 10am<br>
→ Basketball, Court B, Tuesday 7pm<br>
→ Volleyball, Court C, Sunday 2pm</p>

<p>The slots feel exclusive because they kind of are — only one person can book each.</p>

<p>Lock yours in: <a href="{{ event.resume_url }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Resume booking</a></p>

<p style="font-style:italic;margin-top:20px;">— The RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you wanted a different time, just reply with the slot you actually want and we'll send a direct booking link.</p>
```

The "what others booked this week" list is hard-coded for now. To make it dynamic, you'd need a daily-updating set of profile properties (e.g., `popular_booking_1`, `popular_booking_2`, `popular_booking_3`) — overkill for v1. Hard-code, refresh quarterly.

---

### Touch 03 — Real scarcity (SMS, conditional on availability)

- **Channel:** SMS
- **Send delay:** Wait `24 hours` after Touch 02 → +48 hours total
- **Sender label:** `RunRec`
- **Send time:** Standard SMS quiet hours apply (Klaviyo default — typically 10 AM – 8 PM recipient timezone for SMS)

#### Body content (SMS)

```
Your {{ event.sport }} court at {{ event.time }} is still free. Tap to grab it: {{ event.resume_url }} -RunRec · Stop=opt-out
```

If `event.resume_url` is too long for SMS, use a URL shortener like `runrec.co/r/{{ event.cart_token }}` — depends on your URL infra.

---

#### THE WEBHOOK CHECK (this is the unique complexity)

Touch 03 must ONLY fire if the slot is still genuinely available. Faking scarcity ("Last chance!") when the slot is actually gone is a credibility-killer. So we need a real-time availability check.

There are two ways to wire this in Klaviyo:

##### Method 1 — Klaviyo Conditional Split + external webhook (recommended)

Before Touch 03 (after the 24-hour wait), drag in a **Conditional Split** (Klaviyo calls it "Conditional Split" or "If/Else").

Set the condition to:

- **Custom property check** — but Klaviyo's conditional splits don't natively make outbound HTTP calls. So you can't do real-time availability check this way directly.

##### Method 2 — Klaviyo Webhook action + property update + Conditional Split

This is the correct architecture. It works like this:

1. After Touch 02 + 24-hour wait, drag in a **Webhook** action.
   - URL: `https://api.therunrec.com/v1/availability/check?cart_token={{ event.cart_token }}`
   - Method: GET (or POST with cart_token in body)
   - The webhook endpoint returns JSON: `{ "available": true }` or `{ "available": false }`
   - **CRITICAL:** Klaviyo's Webhook action needs to write the result back to the profile. Configure the webhook to update a profile property called `last_cart_check_available` with the response value.

   Actually — Klaviyo's native Webhook action sends data OUT but doesn't always parse responses inbound. You may need to flip this: the webhook endpoint, when called, fires a Klaviyo `availability_checked` event back into Klaviyo with `{ cart_token, available }`. The flow waits for that event before deciding.

   This gets complex. **Simpler alternative for v1:**

##### Method 3 — Skip the real-time check, fire Touch 03 with safety language (RECOMMENDED FOR LAUNCH)

For v1: skip the webhook check entirely. Use safety-language SMS:

```
Just checking — your {{ event.sport }} court for {{ event.time }} might still be open. Check: {{ event.resume_url }} -RunRec · Stop=opt-out
```

This avoids the credibility problem of saying "your slot is free!" when it's not. Less aggressive than the planned scarcity message but ships now.

When Eyad/Rob build the availability webhook, swap Method 3 for Method 2 in v2.

#### Klaviyo UI walkthrough for Touch 03 (Method 3 — recommended for launch)

1. After Touch 02, drag Time Delay → 24 hours.
2. Click **"+"** → SMS.
3. Name: `Flow 12 — T03 — Real Scarcity SMS`.
4. Sender Profile: `RunRec`.
5. Body content from Method 3 above (the safer language).
6. Smart Sending: OFF.
7. Filter: SMS Marketing subscribed.
8. Save.

#### When the webhook is built (Method 2, future)

1. After Touch 02, drag Time Delay → 24 hours.
2. Drag in **Webhook** action.
3. Configure URL, method, and how the response is captured (consult Klaviyo docs and your webhook implementation).
4. Wait 5 minutes for response (use a Time Delay).
5. Drag in **Conditional Split**.
6. Condition: profile property `last_cart_check_available` equals `true`.
7. YES path → Touch 03 SMS (the aggressive scarcity copy).
8. NO path → exit flow.

---

## Test plan

Before activating:

1. **Verify the trigger event fires:** have dev fire a test `booking_abandoned` event for your test profile. Check Klaviyo Activity → confirm event received.
2. **Verify event payload merge tags resolve.** Open Touch 01 in preview, point at the test event. Confirm `{{ event.sport }}`, `{{ event.court_letter }}`, `{{ event.date }}`, `{{ event.time }}`, `{{ event.total }}`, `{{ event.resume_url }}` all show real values.
3. **Test Touch 01 timing:** fire the trigger and wait. Touch 01 should arrive within 1 hour ± 5 min.
4. **Test the exit condition:** fire trigger, then within 12 hours fire a `Successfully Paid` event with matching `cart_token`. Verify Touch 02 does NOT fire.
5. **Test re-entry filter:** fire two `booking_abandoned` events for the same profile within 7 days. Verify only the first one triggers the flow.
6. **Test SMS Touch 03:** receive on real phone. Verify URL clicks open resume booking.
7. **CRITICAL: verify Browse Abandoned flows are paused.** Check `RncQfn` and `WajxZ4` in Klaviyo Flows list — both should be in DRAFT, not LIVE. If still LIVE, customers will get TWO flows simultaneously.

---

## Activation checklist

- [ ] `booking_abandoned` custom event firing from Vercel checkout (verified in Klaviyo Metrics list)
- [ ] At least 5 successful test events recorded in Klaviyo
- [ ] All 3 touches built
- [ ] Touch 03 webhook decision documented (Method 2 with webhook OR Method 3 simpler language)
- [ ] **Browse Abandoned flows `RncQfn` and `WajxZ4` PAUSED** (drafted in Klaviyo)
- [ ] Smart Sending OFF on all touches
- [ ] Quiet hours OFF on Touch 01 + Touch 02
- [ ] Quiet hours ON for Touch 03 SMS (Klaviyo default)
- [ ] Sender Profile `RunRec` exists for SMS
- [ ] Event payload merge tags verified rendering correctly in test preview
- [ ] Resume URL is a working deep link (not 404)
- [ ] Inbox triage assigned for Touch 01 PS replies ("if something didn't work in checkout, just reply")
- [ ] Operator approval

---

## Post-launch monitoring (first 14 days — DAILY)

This flow is high-volume + revenue-critical. Monitor closely.

- **Trigger volume.** How many `booking_abandoned` events fired? Should match Vercel checkout abandonment rate (typically 30-50% of checkout sessions).
- **Touch 01 deliverability.** Target 95%+ delivered. Below 90% = sender domain issue or recipient ISP throttling (high-volume new flow can trip filters).
- **Touch 01 open rate.** Target 50-65%. These are people in active intent — should be the highest open rate of any flow.
- **Touch 01 click-to-resume rate.** Target 15-25%. This is the recovery moment.
- **Recovery rate (entered flow → completed `Successfully Paid`).** Target 15-25% per the buildsheet. Track at 24h, 48h, 72h checkpoints.
- **RPR (revenue per recipient).** Target $3.65 average per the buildsheet. This is the highest-RPR flow in the system.
- **Touch 02 open rate.** Target 35-45%. If higher than Touch 01, the subject line on T01 is wrong.
- **Touch 03 fire rate.** What % of profiles reach Touch 03? Should be 50-60% (others either booked or unsubscribed by then).
- **Browse Abandoned overlap check.** Run a Klaviyo report: any profile receiving BOTH `RncQfn` and Flow 12 in last 24 hours = critical bug. Should be ZERO.

---

## Common issues for THIS flow

**Issue: Trigger doesn't fire.**
- `booking_abandoned` event isn't being sent from Vercel. Check with Eyad/Rob — confirm Klaviyo Track API call is in the Vercel checkout abandonment handler.
- Metric not registered in Klaviyo. Need to fire one test event for it to appear in Metrics list.

**Issue: Email shows blank merge tags (`{{ event.sport }}` literally in the email).**
- Event payload doesn't include the field. Check the JSON being sent — does the event properties object include `sport`?
- Or: merge tag syntax wrong. Klaviyo uses `{{ event.PROPERTY }}` for event-driven flows; verify exact syntax.

**Issue: Customer who completed booking still receives Touch 02.**
- Exit condition isn't matching cart_token. Check exit configuration.
- Or: completed booking didn't fire `Successfully Paid` with matching cart_token. Stripe may not pass the token through — investigate payload mapping.

**Issue: Touch 03 sends but slot was already booked.**
- You're using Method 3 (no webhook check) — this is acceptable since the SMS language is deliberately soft ("might still be open"). If you upgraded to Method 2, the webhook isn't returning the right value — investigate endpoint.

**Issue: Browse Abandoned + Flow 12 firing simultaneously.**
- You forgot to pause `RncQfn` and `WajxZ4`. STOP. Pause them now.

**Issue: High unsubscribe rate (>1% per touch).**
- Cart abandonment volume is high — flow is sending to too many people too fast. Add audience exclusion: skip if profile already unsubscribed from any flow in last 30 days.
- Or: copy is too aggressive. Soften Touch 03 SMS.

**Issue: Multiple abandoned carts from same person inflates send count.**
- 7-day re-entry filter should catch this. Verify it's set correctly.

---

## Template note

All emails use **Flow / Personal**. Footer required:
- `RunRec` · `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8` · unsubscribe link · manage preferences link

This flow's emails should be VERY plain — text-forward, single-image-max. Cart-abandonment emails that look "promotional" get filtered. Plain-text-feel beats fancy.
