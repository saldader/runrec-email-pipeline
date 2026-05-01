# Flow 02 — Booking Confirm + Reminders

**Build time estimate:** ~1 hour
**Difficulty:** Medium (4 touches, but the trigger logic is the hard part)
**Prerequisites:**
- Pre-flight setup must be complete (see `~/vault/runrec/marketing/klaviyo-implementation/00-current-klaviyo-state.md`)
- Sender domain `contact@therunrec.com` authenticated (DKIM/SPF/DMARC)
- "Transactional" master template should exist (see specs at end of this doc)
- Stripe → Klaviyo integration is live (it is — `Successfully Paid` event is firing). Confirm in Settings → Integrations.
- Profile properties `last_booking_date` and `last_sport_booked` exist (string + date) — Flow 02 writes to these
- The booking system writes the following properties to the `Successfully Paid` event:
  - `sport` — e.g., "Basketball" or "Pickleball"
  - `court_letter` — e.g., "A," "B," or "C"
  - `date` — booking date as a string the merge tag can render
  - `time` — booking time as a string
  - `booking_end_time` — used to calculate when Flow 03 fires
- **Critical decision before you start:** Skedda is the booking system. Skedda also sends its own confirmation emails. **Decide:** are you (a) replacing Skedda's emails entirely with Klaviyo, or (b) layering Klaviyo on top? If (b), confirm that customers won't get two confirmation emails. The cleanest answer is (a) — turn off Skedda's email sends and let Klaviyo own the lifecycle.

**Klaviyo glossary** (terms used in this guide):
- **Metric trigger** — a flow trigger that fires when a specific event happens (e.g., `Successfully Paid` from Stripe), as opposed to a list trigger (Flow 01 uses a list trigger).
- **Conditional split** — a fork in the flow where Klaviyo checks a condition and routes the profile down one path or another.
- **Time delay relative to event** — wait until X hours/days before or after a specific timestamp in the event payload (e.g., "send 24 hours before `date`"). Klaviyo calls this a "Wait until" action.

---

## What this flow does (plain English)

When a customer pays for a court booking, this flow sends them 4 messages: an email confirmation immediately (with the booking details and a soft upsell to birthday packages), a quick text confirmation 5 minutes later (so they know to expect the day-of texts), an email checklist 24 hours before the booking (what to bring, parking, doors), and a final SMS reminder 2 hours before the booking with the door access code.

The flow exists for one reason: **reduce no-shows by 20–30%.** A booking that gets a real confirmation + 24-hour email + 2-hour text shows up. A booking that gets only Skedda's auto-receipt forgets they booked. The 4-touch sequence pays for itself in saved no-show inventory.

---

## Trigger — what fires this flow

- **Type:** Metric (event-based)
- **Event:** `Successfully Paid` (from Stripe integration)
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name the flow: `Flow 02 — Booking Confirm + Reminders (v2 · 2026-05)`
  3. Trigger type: **Metric**
  4. Choose metric: **Successfully Paid** (Stripe)
  5. Trigger filter on the metric event: `event_type` equals `court_booking` (this filters out membership renewals, party deposits, etc. — only court bookings should fire this flow). If `event_type` isn't a property on the Stripe event, talk to Eyad about adding it. As an interim, you can filter by `amount` greater than 0 + filter out subscription events, but `event_type` is the clean answer.

---

## Filters

Add these filters on the trigger:

- **Has valid email** — Klaviyo: "Properties about someone → Email → is set."
- **Has valid phone number with SMS consent** — needed for Touches 02 + 04. Profiles without SMS consent will skip the SMS touches but should still get the email touches. This filter goes on the SMS touches individually (see touch instructions), NOT at the trigger level — at the trigger level just require email.
- **Booking date is in the future** — Klaviyo: "Trigger event property `date` is on or after today." This prevents the flow from firing on backdated bookings, refunds, or admin corrections.
- **Has not been in this flow for the same booking** — Klaviyo's "Has been in this flow" check works per-trigger-event, so this is automatic. No extra config needed unless you see double sends in testing.

---

## Exit conditions

Click the trigger → "Flow exits" tab → add:

- **Booking cancelled or refunded** — Klaviyo: "Has done event → Refunded Payment → at least once since starting this flow." Profile exits without further touches.
- **Booking date has passed** — Klaviyo automatically completes the flow once Touch 04 (T-2hrs) sends. No extra config needed.

---

## Smart Sending: **OFF** — and why

This is a transactional flow. Smart Sending must be OFF or customers will silently miss their confirmation, checklist, or door code if they happen to be in another flow at the same time. Transactional sends are CASL- and CAN-SPAM-exempt from frequency caps for this exact reason — never throttle them.

How to turn off: settings gear icon at the top of each touch → uncheck Smart Sending.

## Quiet Hours: SMS handling

For Touch 02 (Booking +5 min): Quiet Hours **OFF** — the customer just paid; they expect immediate confirmation. The 5-minute SMS lands within seconds of the email.

For Touch 04 (T-2 hours): Quiet Hours **ON**, 8am–9pm recipient local. If the booking is at 9am, the T-2hrs SMS would fire at 7am — Quiet Hours pushes it to 8am instead. If the booking is at 11pm, T-2hrs is 9pm — fires at 9pm sharp, just inside the window. If the booking is at 1am (yes, RunRec is 24/7), T-2hrs is 11pm and Quiet Hours pushes it to 8am the next day — too late. **For any booking between 11pm and 9am local, manually skip Touch 04** by adding a conditional split (see Touch 04 instructions).

---

## Flow map (visual)

```
TRIGGER: Successfully Paid event (Stripe), event_type = court_booking
   ↓
   (Filter: future booking, valid email)
   ↓
Touch 01 EMAIL — Booking +0, immediate — "You're locked in. Here's everything."
   ↓
Wait 5 minutes
   ↓
   Conditional: does profile have SMS consent?
   ├── YES → Touch 02 SMS — Quick text confirmation
   └── NO → skip Touch 02, continue to wait step
   ↓
Wait until 24 hours before booking date/time
   ↓
Touch 03 EMAIL — T-24 hours — "Tomorrow's checklist."
   ↓
Wait until 2 hours before booking date/time
   ↓
   Conditional: is the local time of T-2hrs between 8am and 9pm?
   ├── YES → Conditional: does profile have SMS consent?
   │            ├── YES → Touch 04 SMS — Door code + final reminder
   │            └── NO → skip
   └── NO (booking is between 11pm and 9am) → skip Touch 04
   ↓
END (booking happens; profile exits and Flow 03 picks up 2 hrs after booking ends)
```

---

## Touch-by-touch build instructions

### Touch 01 — Full confirmation + soft upsell

- **Channel:** Email
- **Send delay:** Immediate
- **From name:** `The RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `You're locked in. Here's everything.`
- **Preview text:** `Court details, door access, parking, what to bring.`
- **Template to use:** Transactional (clean, no hero image, info block at the top)

**Body content (paste this HTML directly):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>You're booked.</p>
<p><strong>{{ event.sport }} · Court {{ event.court_letter }} · {{ event.date }} at {{ event.time }}</strong></p>
<p>Address: 283 Northfield Dr. E Unit 13, Waterloo<br>
Parking: Free, lot to the left of the building<br>
Bring: Indoor shoes, water, Bluetooth speaker if you want music</p>
<p>The door access code arrives by SMS 2 hours before your booking time — keep your phone nearby.</p>
<p>If anything changes, just reply.</p>
<p style="font-style:italic;">— The RunRec team</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Hosting a party? Our birthday packages already include a coordinator and full setup. Reply if you want package details.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Email** action right after the trigger.
2. Click "Configure content" → fill From label, From email, Subject, Preview text exactly as above.
3. Click "Edit content" → switch the body block to "Custom HTML" and paste the body above.
4. The merge tags `{{ event.sport }}`, `{{ event.court_letter }}`, `{{ event.date }}`, `{{ event.time }}` reference properties on the trigger event. If your Stripe payload uses different property names (e.g., `event.line_items.0.metadata.sport`), substitute. Test with a real Successfully Paid event to verify.
5. Save → Smart Sending: OFF.

**Conditional logic:** None.

**Note on doctrine drift:** The deployed dashboard for this flow says "The door access code arrives by SMS 2 hours before your booking time." The master buildsheet (Section 04 → Flow 02 mechanic note) says doors auto-unlock 5 minutes before each booking and there is no door code at all. **This is a content inconsistency that the operator must resolve before launch.** Two options:
- Option A (current Klaviyo dashboard): use door code language as written above.
- Option B (master buildsheet doctrine): replace the access-code line with `Doors automatically unlock 5 minutes before your booking time — no codes needed.`

Pick one. Don't ship both. **This guide ships Option A as the default because that's what the deployed dashboard shows; flag for operator approval.**

---

### Touch 02 — Quick text confirmation (SMS)

- **Channel:** SMS
- **Send delay:** Wait 5 minutes after Touch 01
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
All set, {{ first_name|default:'there' }} — see you {{ event.date }} at {{ event.time }}. Court {{ event.court_letter }}. Door code arrives 2 hrs before your time.

-RunRec Stop=opt-out
```

(Total under 160 characters with personalization. The Stop=opt-out is the compact CASL-compliant opt-out language.)

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay** action: "Wait 5 minutes."
2. Drag a **Conditional Split** action: condition = "Properties about someone → SMS Consent → equals → Subscribed."
   - YES branch → drag SMS action.
   - NO branch → leave empty. The flow continues past the split to the next step.
3. Configure the SMS action:
   - Sender ID: RunRec
   - Body: paste exactly as above.
   - Quiet Hours: OFF (this is transactional confirmation)
   - Smart Sending: OFF
4. Save.

**Reply routing:** Standard. STOP unsubscribes (Klaviyo handles automatically). Other replies route to the SMS Conversations inbox (Eyad or Sal monitors). Most replies will be questions about the booking — same-day responsiveness matters.

---

### Touch 03 — Pre-session checklist

- **Channel:** Email
- **Send delay:** Wait until 24 hours before `event.date` + `event.time`
- **From name:** `The RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Tomorrow's checklist.`
- **Preview text:** `3 things to bring. Door code times. Parking notes.`
- **Template to use:** Flow / Personal (text-forward, no hero image)

**Body content:**

```html
<p>Quick reminder — you're booked tomorrow at <strong>{{ event.time }}</strong>.</p>
<p>Three things to bring:</p>
<p><strong>1. Indoor shoes</strong> (no street shoes on the hardwood)<br>
<strong>2. Water</strong><br>
<strong>3. Bring your own device — connect to our Bluetooth speakers in the gym</strong> if you want music — the acoustics are unreal</p>
<p>Doors auto-unlock 5 min before your booking. No code needed: arrives by SMS 2 hours before.<br>
Parking: lot to the left of the building, free.<br>
Address: 283 Northfield Dr. E Unit 13, Waterloo</p>
<p>Anything else, just reply.</p>
<p style="font-style:italic;">— The RunRec team</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Hosting a party? Our birthday packages include a coordinator who arrives 30 min early to set up. Reply for package details.</p>
```

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02 (or after the conditional split rejoins), drag a **Time Delay** action.
2. Choose: "Wait until specific time relative to event property."
3. Property: `date` (or `start_time` — whatever your Stripe payload uses for booking start).
4. Offset: -24 hours.
5. Drag an **Email** action after the delay. Configure From/Subject/Preview as above. Paste body content.
6. Smart Sending: OFF.

**Note on doctrine drift:** The body above contains BOTH "Doors auto-unlock 5 min before your booking" AND "No code needed: arrives by SMS 2 hours before." That's contradictory. Same operator decision as Touch 01: pick one mechanic and remove the other.

---

### Touch 04 — Door code + final reminder (SMS)

- **Channel:** SMS
- **Send delay:** Wait until 2 hours before `event.date` + `event.time`
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Court {{ event.court_letter }} in 2 hrs. Doors auto-unlock 5 min before your booking. No code needed: {{ event.access_code }}. See you on the court.
```

(Personalized. Under 160 characters. Note: same doctrine inconsistency — body says doors auto-unlock AND provides an access code. Resolve per Touch 01.)

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay** action: "Wait until 2 hours before event.date."
2. Drag a **Conditional Split**: condition = "Trigger event property `time` is between 10:00 AM and 11:00 PM" (this ensures Touch 04 only fires for daytime bookings — late-night bookings would have Touch 04 land outside Quiet Hours).
   - YES → continue to next conditional.
   - NO → skip to END.
3. Drag another **Conditional Split**: condition = "SMS Consent equals Subscribed."
   - YES → SMS action.
   - NO → skip to END.
4. Configure the SMS action:
   - Sender ID: RunRec
   - Body: paste exactly as above
   - Quiet Hours: ON, 8am–9pm
   - Smart Sending: OFF
5. Save.

---

## Test plan

1. Trigger a real test booking through Skedda for yourself — pick a slot 25 hours in the future so you can verify all 4 touches.
2. Within 5 minutes:
   - Touch 01 email arrives. Verify all merge tags resolve (sport, court letter, date, time).
   - Touch 02 SMS arrives ~5 min later. Verify same merge tags.
3. 24 hours before the booking, Touch 03 email arrives. Verify the time merge tag is correct.
4. 2 hours before the booking, Touch 04 SMS arrives. Verify the access_code merge tag resolves (or is removed if you're using auto-unlock language).
5. After the booking time passes, verify the profile exits the flow cleanly and Flow 03 (Post-Session) picks up 2 hours after `booking_end_time`.

**Edge cases to test:**
- Same-day booking (booking is in 3 hours): Touch 03 should be skipped because there's no 24-hour window.
- Late-night booking (1am): Touch 04 should be skipped per the conditional split.
- Profile with no SMS consent: Touches 02 and 04 should be skipped, but Touches 01 and 03 should still send.
- Refunded booking: profile should exit the flow and not receive any remaining touches.

---

## Activation checklist

- [ ] All 4 touches built, merge tags verified against the actual Stripe payload property names
- [ ] Footer on every email contains business name, physical address, unsubscribe link, manage preferences link
- [ ] Smart Sending: OFF on every touch
- [ ] Quiet Hours: ON for Touch 04 SMS, OFF for Touch 02 SMS
- [ ] Conditional splits wired correctly (SMS consent check on both SMS touches; daytime check on Touch 04)
- [ ] Trigger filter `event_type = court_booking` is set (or equivalent) — flow should NOT fire for membership renewals or party deposits
- [ ] Operator has decided the door code vs auto-unlock doctrine and the copy reflects it consistently across Touches 01, 03, and 04
- [ ] Skedda's auto-confirmation email is either turned off OR the operator has confirmed it's OK that customers receive both
- [ ] Test booking has been made and all 4 touches verified end-to-end
- [ ] Reply inbox monitoring is set up (Eyad/Sal know to watch for booking-related replies)

---

## Post-launch monitoring (first 7 days)

- **Touch 01 open rate:** Target 80%+ (transactional confirmations are the highest-open emails on Earth). Below 60% = something is wrong with deliverability or the booking itself failed.
- **Touch 02 SMS delivery rate:** Target 99%+. Below 95% = phone number quality problem.
- **Touch 03 open rate:** Target 50%+. This is a true reminder; people open it because they have a booking tomorrow.
- **Touch 04 SMS delivery rate:** Target 99%+.
- **No-show rate vs. pre-flow baseline:** This is the real metric. Target: 20–30% reduction in no-shows after 30 days. If no-shows don't move, the flow isn't working — investigate whether customers are actually getting Touch 04 (often a quiet-hours misconfig or SMS consent gap).
- **Reply volume to Touch 01:** Some replies are normal ("can I change the time?"). High reply volume (>5%) usually means the confirmation isn't clear about something — read the replies, refine the copy.

**Warning signs:**
- Open rate on Touch 01 below 50% = check Stripe → Klaviyo integration; events may not be firing properly.
- High unsubscribe rate on transactional touches (above 0.5%) = the flow is being misperceived as marketing — usually the soft-upsell PS is too aggressive. Tone down or remove.
- SMS complaint rate above 0.1% = SMS consent isn't really opt-in; review your collection method.

---

## Common issues for THIS flow

1. **Door code vs auto-unlock doctrine inconsistency.** The single biggest issue with this flow as currently spec'd. The dashboard copy uses door code language; the master buildsheet says no codes, doors auto-unlock. Pick one before you ship. If RunRec's actual access system auto-unlocks (operator confirms), use the auto-unlock language and remove all references to "door access code" / `{{ event.access_code }}`.

2. **Skedda double-confirmation.** Skedda sends its own email. Without intervention, customers get TWO confirmation emails 30 seconds apart. Decide:
   - Option A: turn off Skedda's confirmation email, let Klaviyo own it (cleanest).
   - Option B: keep both, accept the redundancy (lazy).
   - Option C: customize Klaviyo's Touch 01 to be a "supplement" email with checklist + upsell rather than a confirmation (changes the strategic value of the flow).

3. **Stripe payload property names.** The merge tags above (`event.sport`, `event.court_letter`, etc.) assume those properties are present on the `Successfully Paid` event. They may not be — Stripe's default payload doesn't include sport/court info. The booking integration (Skedda → Stripe → Klaviyo) needs to write these as metadata. If they're missing, the merge tags render as blank — looks broken. Test with a real booking and verify.

4. **Late-night bookings (11pm–9am).** RunRec is 24/7. A 2am booking has Touch 04 firing at midnight, which is outside Quiet Hours. The conditional split in Touch 04 above handles this by skipping the SMS — but that means a 2am booker doesn't get a 2-hour reminder, which is the most important touch. Operator decision: is it better to (a) skip entirely (current spec), (b) send the SMS at 9pm the previous day with "your booking is in [X] hours" framing, or (c) violate Quiet Hours for booking-related messages (legally permissible because it's transactional)? Cleanest answer is (c) — turn Quiet Hours OFF for Touch 04 entirely, since it's transactional. But operator should sign off.

5. **The flow fires on every Successfully Paid event.** That includes membership renewals, party deposits, league fees, etc. The trigger filter `event_type = court_booking` is critical. If that property doesn't exist in the Stripe payload, work with Eyad/Rob to add it. Without the filter, this flow will send "court details" emails to people who paid for a membership renewal, looking broken.

6. **Touch 01 PS soft upsell can drift over time.** "Hosting a party?" is currently the upsell. If RunRec adds a new offering (memberships, leagues, corporate packages), the temptation will be to add more upsells to this PS. Don't. One PS upsell per touch. If you want to add memberships, build a separate post-session flow for it.

7. **Existing live `VwYTDc` Post Purchase flow conflict.** The current Klaviyo account has a live "2025 - Post Purchase (Visual Email)" flow that triggers on Successfully Paid. That's actually Flow 03 territory (post-session), but it could conflict with this Flow 02 if both fire on the same Successfully Paid event. Recommended: pause `VwYTDc` (and its SMS counterpart `TXW8MN`) before launching Flow 02. Operator decides whether to archive or rebuild as Flow 03.
