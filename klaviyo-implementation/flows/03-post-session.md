# Flow 03 — Post-Session + Review

**Build time estimate:** ~1 hour
**Difficulty:** Medium (the trigger calculation and the conditional reply routing are the tricky parts)
**Prerequisites:**
- Pre-flight setup must be complete
- Sender domain `contact@therunrec.com` authenticated
- "Flow / Personal" master template exists
- Stripe → Klaviyo integration is live (`Successfully Paid` event firing)
- The booking system writes `booking_end_time` (or equivalent) to the `Successfully Paid` event metadata. Without this, you cannot calculate "2 hours after session ends." Talk to Eyad/Rob to confirm.
- Profile property `last_review_disposition` (string) exists — this guide uses it to track 👍 vs 👎 responses.
- Google review short link `runrec.co/rev` exists and routes to your Google Business Profile review page.
- The existing live "2025 - Post Purchase (Visual Email)" flow `VwYTDc` and SMS counterpart `TXW8MN` should be **paused** before launching this flow, otherwise customers get duplicate post-purchase touches.

**Klaviyo glossary** (terms used in this guide):
- **Wait until event property** — Klaviyo time-delay action that waits until a specific timestamp in the trigger event payload, plus an offset (e.g., "2 hours after `booking_end_time`").
- **Profile property check** — a conditional split that branches based on a profile property value (used here for the 👍/👎 routing).

---

## What this flow does (plain English)

When a customer's session ends, this flow waits 2 hours (long enough to be home and feeling good, short enough that the dopamine hasn't worn off), then sends a 2-emoji SMS asking "How was it? 👍 or 👎." Thumbs-up routes to a Google review nudge SMS 30 minutes later. Thumbs-down routes to a private feedback form (NEVER a public review). Either way, 3 days later the flow sends an email with specific open slots for re-booking. If they still haven't rebooked after 7 days, a real-scarcity SMS lands.

The strategic point: the 2-hour post-session window is when Google reviews and rebookings actually happen. After 24 hours the magic fades and you're competing with everything else in their inbox. This flow strikes the iron while it's hot.

---

## Trigger — what fires this flow

- **Type:** Metric (event-based)
- **Event:** `Successfully Paid` (Stripe) — same event as Flow 02, but the trigger fires the flow at a different time.
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name the flow: `Flow 03 — Post-Session + Review (v2 · 2026-05)`
  3. Trigger type: **Metric**
  4. Choose metric: **Successfully Paid**
  5. Trigger filter: `event_type` equals `court_booking` (same as Flow 02 — exclude membership/party events)

**Important note on trigger timing:** Flow 02 fires immediately on `Successfully Paid`. Flow 03 also fires on `Successfully Paid`, but the FIRST action in Flow 03 is a "Wait until 2 hours after `event.booking_end_time`" step. So both flows trigger at booking, but they execute touches at different times. There's no conflict.

If your Stripe event doesn't carry `booking_end_time` as a property, you have two options:
- (a) Calculate it: `event.date` + `event.time` + `event.duration_hours` + 2hrs. Klaviyo can do this with a custom delay configured in JS-style if you have the Klaviyo API helper extensions.
- (b) Use a 2-day fixed delay as a proxy. Less precise but works as an interim.

Option (a) is correct. Get Eyad to add `booking_end_time` as a metadata field if it doesn't already exist.

---

## Filters

Add to the trigger:

- **Has not received Post-Session flow in last 7 days** — Klaviyo: "Was in this flow → Zero times in the last 7 days." Prevents over-mailing if a customer books two sessions in one week.
- **Has valid email** — Klaviyo: "Properties about someone → Email → is set."

---

## Exit conditions

- **Books another session** — Klaviyo: "Has done event → Successfully Paid → at least once since starting this flow." Profile exits remaining touches (no need to nudge them to rebook if they already did).
- **Profile property `last_review_disposition` equals `negative`** — exit before Touch 02 (Google review nudge) so we never ask a thumbs-down customer to publicly review.
- **Unsubscribes** — automatic.

---

## Smart Sending: **ON for Touches 03 and 04, OFF for Touches 01 and 02**

Touches 01 (NPS thumbs check) and 02 (Google review nudge) are time-sensitive and should always send. Touches 03 (calendar of slots) and 04 (scarcity SMS) are softer asks and can be skipped if the customer just received another email/SMS.

How: settings gear icon on each touch → toggle Smart Sending per touch.

## Quiet Hours: **ON for all SMS**

All three SMS touches in this flow respect Quiet Hours. Window: 8am–9pm recipient local timezone. Klaviyo holds late-night sends until morning.

---

## Flow map (visual)

```
TRIGGER: Successfully Paid event, event_type = court_booking
   ↓
   (Filter: not in flow recently, valid email)
   ↓
Wait until 2 hours after event.booking_end_time
   ↓
Touch 01 SMS — "How was it? 👍 or 👎"
   ↓
   Conditional: profile property last_review_disposition equals "positive"?
   │   (this gets set when they reply 👍 — see reply routing below)
   ├── YES → Wait 30 minutes → Touch 02 SMS (Google review nudge) → continue
   ├── NO (i.e., 👎 or no response) →
   │       ├── if last_review_disposition = "negative" → EXIT (private feedback path)
   │       └── if no response → continue to Touch 03 directly
   ↓
Wait 3 days
   ↓
   Conditional: has profile booked since starting this flow?
   ├── YES → EXIT (already rebooked, don't nudge)
   └── NO → Touch 03 EMAIL — Calendar of open slots
   ↓
Wait 4 more days (total 7 days from session end)
   ↓
   Conditional: still no booking?
   ├── YES (still no booking) → Touch 04 SMS — Loss-aversion scarcity nudge
   └── NO → EXIT
   ↓
END
```

---

## Touch-by-touch build instructions

### Touch 01 — NPS-lite thumbs check (SMS)

- **Channel:** SMS
- **Send delay:** Wait until 2 hours after `event.booking_end_time`
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
How was it? 👍 or 👎 Stop=opt-out
```

(Total: 33 characters with opt-out. Five words. Two emojis. The simplest possible ask.)

**Klaviyo UI walkthrough — Touch 01:**
1. After the trigger, drag a **Time Delay** action.
2. Configure: "Wait until specific time relative to event property → `booking_end_time` → +2 hours."
3. Drag a **Conditional Split**: condition = "SMS Consent equals Subscribed."
   - YES → continue.
   - NO → skip to Touch 03 directly (no thumbs SMS, no review nudge — but they still get the calendar email later).
4. Drag an **SMS** action after the consent check.
5. Sender ID: RunRec. Body: paste exactly as above.
6. Quiet Hours: ON. Smart Sending: OFF (always send).
7. Save.

**Reply routing (set up via Klaviyo SMS Conversations or via webhook):**
- 👍 reply → set profile property `last_review_disposition` = `positive`. Trigger continues to Touch 02.
- 👎 reply → set profile property `last_review_disposition` = `negative`. Send a private feedback email/form to the customer (separate side flow or manual escalation to Sal/Izzy). Profile exits Flow 03.
- No reply → no property update. Profile continues to Touch 03 in 3 days (skipping Touch 02).
- STOP → standard unsubscribe handling.

To configure: Klaviyo's SMS Conversations doesn't natively branch flows on emoji replies. Easiest approach is:
- Use Klaviyo's "Subscribe to webhook" or Zapier to listen for inbound SMS.
- The webhook parses the reply (👍 emoji or 👎 emoji or text) and updates the profile via Klaviyo API.
- The conditional split below then branches on the now-updated property.

If webhook setup is too complex, fallback: skip the branching. Send Touch 02 (Google review nudge) to everyone 30 minutes after Touch 01, regardless of response. This loses the thumbs-down protection but is much simpler. **Operator decision** — recommend the webhook approach long-term but it's OK to launch with the simpler version and improve later.

---

### Touch 02 — Google review nudge (SMS, thumbs-up only)

- **Channel:** SMS
- **Send delay:** Wait 30 minutes after Touch 01, AND profile property `last_review_disposition` equals `positive`
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
We'd love to hear from you. Quick one — would you mind dropping a line on Google? Helps us a lot: runrec.co/rev

-Sal
```

(Total: under 160 chars. Conversational ask. Signed Sal — feels personal.)

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay** action: "Wait 30 minutes."
2. Drag a **Conditional Split**: condition = "Profile property `last_review_disposition` equals `positive`."
   - YES → SMS action below.
   - NO → skip to next step (continues to the 3-day wait before Touch 03).
3. Drag an **SMS** action.
4. Sender ID: RunRec. Body: paste exactly.
5. Quiet Hours: ON. Smart Sending: OFF.
6. Save.

**Reply routing:** Standard — STOP unsubscribes; other replies route to monitoring inbox. Customers who actually click `runrec.co/rev` are the conversion. Monitor click-through rate as the key metric for this touch.

---

### Touch 03 — Calendar of open slots

- **Channel:** Email
- **Send delay:** Wait 3 days after Touch 01 (i.e., 3 days + 2 hours after `booking_end_time`)
- **From name:** `The RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `These slots are still open this week.`
- **Preview text:** `4 specific times. One click each.`
- **Template to use:** Flow / Personal
- **Hero image:** Visual calendar grid showing the 7-day forecast, with the 4 highlighted slots marked in accent orange. OR text-forward layout with each slot as a clickable card. Calendar visual is preferred but text-forward also works.

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Your dopamine from {{ event.date }}'s session is fading. Time to lock in the next one.</p>
<p>Here's what's open this week:</p>
<p>→ Tuesday 8pm · Court A<br>
→ Wednesday 10pm · Court B<br>
→ Thursday 7am · Court C ($45/hr early bird)<br>
→ Friday 9pm · Court A</p>
<p>Pick one: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Book within 48 hours and you'll auto-slot into our regulars list — early access to new programs, occasional surprise hours on the house. No fee to opt in.</p>
```

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02 (or after Touch 02 was skipped), drag a **Time Delay** action: "Wait 3 days."
2. Drag a **Conditional Split**: condition = "Has done event → Successfully Paid → at least 1 time since starting this flow."
   - YES → EXIT flow.
   - NO → continue to email.
3. Drag an **Email** action.
4. Configure From/Subject/Preview/Body as above.
5. Smart Sending: ON.
6. Save.

**Note on the 4 hard-coded slot times:** Tuesday 8pm, Wednesday 10pm, Thursday 7am, Friday 9pm — these are placeholder examples. Ideally the calendar uses live availability data via a Klaviyo dynamic block fed by an API. Without that, the operator has two choices:
- Option A: hard-code 4 representative slots (current spec) and accept that some recipients will see slots that no longer exist when they click.
- Option B: send a single CTA "Book your next session" with no specific slots — less effective but never lies.

Operator should choose. Long-term, build the dynamic-block integration.

---

### Touch 04 — Loss-aversion nudge (SMS)

- **Channel:** SMS
- **Send delay:** Wait 4 more days after Touch 03 (total 7 days after session)
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Quick heads up — only 3 evening slots left this week. runrec.co/wk
```

(Under 80 characters. Real availability data only — see common issues below.)

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay** action: "Wait 4 days."
2. Drag a **Conditional Split**: condition = "Has done event → Successfully Paid → at least 1 time since starting this flow."
   - YES → EXIT.
   - NO → continue.
3. Drag a **Conditional Split**: condition = "SMS Consent equals Subscribed."
   - YES → SMS action.
   - NO → EXIT.
4. Configure SMS:
   - Sender ID: RunRec
   - Body: paste exactly
   - Quiet Hours: ON
   - Smart Sending: ON
5. Save.

**Critical:** The body says "only 3 evening slots left this week." This is a SCARCITY claim. If it's not true, it's a violation of CASL and Hormozi's "scarcity only works when it's true" principle. The link `runrec.co/wk` should ideally check live availability before redirecting. If the link can't do that, operator should rewrite the body to a true statement, e.g., `Quick heads up — this week's calendar is filling fast. runrec.co/wk` (still creates urgency without lying).

---

## Test plan

1. Trigger a real test booking through Skedda for yourself, scheduled to end ~2 hours from now.
2. After session end + 2 hours, Touch 01 SMS arrives. Verify it landed at the right time and reads correctly.
3. Reply 👍 from your phone. Verify within 30 minutes:
   - Profile property `last_review_disposition` updates to `positive` (check in Klaviyo profile view).
   - Touch 02 SMS arrives with the Google review link.
4. Click the `runrec.co/rev` link. Verify it routes to your Google Business Profile review page.
5. Wait 3 days (or skip ahead in Klaviyo). Touch 03 email arrives if you haven't booked again. Verify subject, preview, body, calendar slot links.
6. Wait 4 more days (or skip ahead). Touch 04 SMS arrives.

**Edge cases to test:**
- Reply 👎 instead of 👍. Verify `last_review_disposition` updates to `negative` and the flow exits before Touch 02.
- No reply at all. Verify Touch 02 is skipped but Touch 03 still fires.
- Book a second session before Touch 03 fires. Verify Touch 03 and 04 are skipped (you exited the flow).
- Profile with no SMS consent — should skip Touches 01, 02, 04 but still receive Touch 03 email.

---

## Activation checklist

- [ ] All 4 touches built
- [ ] Trigger event filter set (`event_type = court_booking`)
- [ ] `booking_end_time` event property exists in the Stripe payload (verified with Eyad)
- [ ] Reply-routing webhook is configured (👍/👎 sets `last_review_disposition`) OR operator has decided to skip the branching for v1
- [ ] Existing `VwYTDc` and `TXW8MN` flows are paused
- [ ] Smart Sending: OFF on Touches 01 + 02, ON on Touches 03 + 04
- [ ] Quiet Hours: ON on all 3 SMS touches
- [ ] `runrec.co/rev` short link verified to route to Google review page
- [ ] `runrec.co/wk` short link verified (live availability ideally)
- [ ] Footer on email touch contains business name, address, unsubscribe, manage preferences
- [ ] Test booking has been through full flow with both 👍 and 👎 paths verified

---

## Post-launch monitoring (first 7 days)

- **Touch 01 reply rate:** Target 25%+ (👍 + 👎 combined). Below 15% means the SMS isn't landing or the timing's off.
- **👍 percentage:** Target 80%+ of replies. Below 70% means there's a service quality issue surfacing through this flow — pause and talk to Sal/Izzy.
- **Touch 02 click-through to `runrec.co/rev`:** Target 30%+. Below 20% means the review nudge copy needs work.
- **Google review velocity:** Track new Google reviews per week before vs after launch. Should see a 2-3× increase in the first 30 days.
- **Touch 03 click-through:** Target 8%+. Below 4% means the slots aren't compelling or the audience is wrong.
- **Touch 04 conversion to booking:** Target 5%+ of recipients book within 48 hours of Touch 04. Below 2% means the scarcity claim isn't credible.

**Warning signs:**
- Spam complaint rate above 0.05% — indicates customers don't think they signed up for marketing. Tighten consent collection.
- Touch 02 driving 1-star Google reviews — means the 👎 routing is broken; thumbs-down customers are receiving the public review nudge. **Pause immediately** and fix the conditional split.
- Touch 04 fake-scarcity backlash — if customers reply asking about "only 3 slots" and there are actually 30, the trust damage is significant. Switch to a true statement.

---

## Common issues for THIS flow

1. **The `booking_end_time` calculation is the single biggest blocker.** Without this property on the Stripe event, you can't trigger Touch 01 at the right time (2 hours post-session). Talk to Eyad/Rob first thing. As an interim, use a fixed 4-hour delay from `Successfully Paid` event time and accept that early-morning bookings will get the SMS at lunchtime.

2. **The 👍/👎 reply routing requires a webhook OR a manual workaround.** Klaviyo doesn't natively branch flows based on inbound SMS content. Three options:
   - (a) Build a webhook (Zapier, Make.com, or custom) that listens for inbound SMS and updates the profile property. Cleanest.
   - (b) Skip branching — send Touch 02 to everyone 30 min after Touch 01. Loses thumbs-down protection.
   - (c) Use a Klaviyo SMS keyword response: configure `THUMBSDOWN` and `THUMBSUP` keywords (less natural but works without a webhook). Customers won't actually type those words though.

   Recommend (a) for production, (b) for v1 launch.

3. **Touch 03 hard-coded slots become stale.** Without dynamic availability, the email shows fixed slots. By the time it sends, those slots may be booked. Acceptable for v1; build dynamic injection in Phase 2.

4. **Touch 04 scarcity copy must be true.** "Only 3 evening slots left" must actually reflect availability. Operator decision: either (a) wire the SMS link to a live-availability check before sending, or (b) rewrite the body to true statements only.

5. **Conflict with the existing `VwYTDc` flow.** That flow currently fires on Successfully Paid with a 2-day delay (close to but not matching this flow's timing). If both are live, customers receive duplicate post-session content. **Pause `VwYTDc` and `TXW8MN` before launching Flow 03.** Do this in Klaviyo: open the flow, set status to "Draft." Existing in-flight profiles complete their current send but new entrants are blocked.

6. **The `last_review_disposition` property persists across bookings.** If a customer gives 👎 in January (sets `last_review_disposition = negative`) and then books again in March, the March flow will see `last_review_disposition = negative` and exit before Touch 02. Two options:
   - (a) Reset the property at flow entry. Add a "Set property" action at the very start: `last_review_disposition = unset`. Cleaner.
   - (b) Accept that one bad experience permanently exempts the customer from review nudges. Possibly more correct, since they may genuinely not want to be asked.
   Operator decision.

7. **The 7-day post-session window may exit prematurely.** If a customer books a recurring weekly slot, the next booking's Successfully Paid event fires within 7 days, exits this flow, and starts a new copy of the flow. That's actually correct behavior — recurring bookers get a fresh review prompt each session. Don't try to "fix" it.
