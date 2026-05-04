# Flow 04 — Customer Birthday

**Build time estimate:** ~1.5 hours
**Difficulty:** Medium-Hard (the trigger is date-based, which Klaviyo handles, but the data collection is the hard part)
**Prerequisites:**
- Pre-flight setup must be complete
- Sender domain `contact@therunrec.com` authenticated
- "Flow / Personal" master template exists
- Profile property `birthday_dates` exists (date type, MM-DD format minimum) — populated for the relevant customers
- Profile property `bday_gift_redeemed_year` exists (numeric) — used to prevent duplicate gifting in same calendar year
- The customer must have placed at least 1 booking in the last 12 months (filter requirement)
- Birthday data must be in Klaviyo. Currently the master buildsheet says: "Manual export from booking history, quarterly process." Until the operator runs that import, this flow has no audience to fire on.
- Decision needed: the master buildsheet says the BDAY-`{{year}}` code mechanic is deprecated as of 2026-05-01 in favor of "screenshot the SMS and email it to redeem." The deployed dashboard still uses the code mechanic in Touch 04 ("Code: BDAY-{{year}}") but Touch 02 uses the screenshot mechanic. **Operator must pick one and apply consistently across Touches 02, 03, 04.** This guide ships the deployed dashboard copy as default but flags the inconsistency.

**Klaviyo glossary** (terms used in this guide):
- **Date-based trigger** — a flow trigger that fires based on a date in a profile property (e.g., birthday) rather than an event. Klaviyo calls this "Date Property Trigger."
- **DOB-7** — shorthand for "7 days before date of birth." Klaviyo's date-property trigger lets you offset by ± days.

---

## What this flow does (plain English)

Seven days before a customer's birthday, this flow sends a heads-up email ("birthday week is coming, we've got something for you"). On the birthday morning at 9am local time, an SMS lands with the gift: a free hour at RunRec, redeemed by screenshotting the SMS and emailing it to the team. Three days later, an email reminds them of the gift and suggests off-peak slots. If they haven't redeemed by Day 14, a deadline reminder email fires. If they still haven't redeemed at Day 28 (final 48 hours), a last-call SMS fires.

The strategic point: **a real gift beats a discount.** People talk about gifts. Nobody talks about 10% off. A free hour costs RunRec near-zero in marginal cost (the court is empty anyway off-peak) but feels like a $89 gift. This is the highest emotional-leverage flow in the system.

---

## Trigger — what fires this flow

- **Type:** Date Property Trigger
- **Property:** `birthday_dates` (the profile's birthday)
- **Offset:** -7 days (fires 7 days BEFORE the birthday)
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name: `Flow 04 — Customer Birthday (v2 · 2026-05)`
  3. Trigger type: **Date Property Trigger**
  4. Property: `birthday_dates`
  5. Offset: `7 days before`
  6. Time of day to fire: 10am recipient local timezone

Note: Klaviyo's Date Property Trigger evaluates daily. Once a profile is in the flow, it will not be re-triggered for 12 months even if the property persists — Klaviyo handles this natively. But add the "not in flow in last 11 months" filter below as a belt-and-suspenders.

---

## Filters

- **Has placed ≥1 booking in last 12 months** — Klaviyo: "Has done event → Successfully Paid → at least 1 time in the last 12 months." This restricts the flow to actual customers, not just newsletter subscribers who happened to share their birthday.
- **Has not received Customer Birthday flow in last 11 months** — Klaviyo: "Was in this flow → Zero times in the last 11 months." Prevents double-sends.
- **Has email marketing consent = subscribed**

---

## Exit conditions

- **Redeems the free-hour offer** — track this however the operator implements redemption. If using the screenshot-and-email mechanic, the team manually marks the profile with `bday_gift_redeemed_year = {current_year}`. Once that property updates, this exit condition fires: "Profile property `bday_gift_redeemed_year` equals current year → EXIT."
- **DOB +28 days passes** — automatic; the flow runs out of touches.
- **Unsubscribes** — automatic.

---

## Smart Sending: **OFF** — and why

Birthday flows are annual. If Smart Sending skips a touch because the customer was in another flow that day, they don't get a second chance for 365 days. Always OFF.

## Quiet Hours: **ON for SMS**

8am–9pm recipient local. Touch 02 is set to fire at 9am — if the recipient is in a different timezone where 9am hasn't arrived yet, Klaviyo holds the send appropriately.

---

## Flow map (visual)

```
TRIGGER: Date Property Trigger, birthday_dates, -7 days, 10am local
   ↓
   (Filter: ≥1 booking in 12mo, not in flow recently, email consent)
   ↓
Touch 01 EMAIL — DOB -7 — "Birthday week is coming."
   ↓
Wait 7 days → 9am recipient local
   ↓
Touch 02 SMS — DOB day, 9am — Birthday gift drop (screenshot to redeem)
   ↓
Wait 3 days → 10am
   ↓
   Conditional: bday_gift_redeemed_year = current year?
   ├── YES → EXIT
   └── NO → Touch 03 EMAIL — DOB +3 — Redemption proximity reminder
   ↓
Wait 11 days → 10am (DOB +14)
   ↓
   Conditional: still unredeemed?
   ├── YES → EXIT
   └── NO → Touch 04 EMAIL — DOB +14 — Mid-window deadline
   ↓
Wait 14 days → 2pm (DOB +28)
   ↓
   Conditional: still unredeemed?
   ├── YES → EXIT
   └── NO → Touch 05 SMS — DOB +28, 48 hrs left — Final hours
   ↓
END
```

---

## Touch-by-touch build instructions

### Touch 01 — Birthday week warmup

- **Channel:** Email
- **Send delay:** Immediate on trigger (DOB -7, 10am local)
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Birthday week is coming.`
- **Preview text:** `Heads up. We've got something for you.`
- **Template to use:** Flow / Personal
- **Hero image:** Wrapped gift box on the hardwood, single overhead light. Mood: anticipation, not garish. Avoid balloon-and-streamers cliché — the brand is "premium private courts," not "kids party shop."

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Your birthday's coming up next week.</p>
<p>We've got something for you. We'll text you the details on the day.</p>
<p>Quick heads up — just so it doesn't go to spam.</p>
<p style="font-style:italic;">— RunRec team</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Don't worry, it's not a 10% off coupon. People talk about gifts, not discounts.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Email** action right after the trigger.
2. Configure From label, From email, Subject, Preview text exactly as above.
3. Edit content → paste body content above.
4. Save → Smart Sending: OFF.

---

### Touch 02 — Birthday gift drop (SMS)

- **Channel:** SMS
- **Send delay:** Wait 7 days, send at 9am recipient local timezone
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Happy birthday {{ first_name|default:'there' }} 🎉 A free hour at RunRec is on us this month. Screenshot this email and reply to redeem. Book yours: runrec.co/bday

-RunRec Stop=opt-out
```

(Personalized. Under 200 characters with personalization. Note: body says "screenshot this email and reply to redeem" — but it's an SMS not an email. Likely a copy artifact. Should read: "Screenshot this message, email it to us, and our team will redeem your free hour." Operator decision.)

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay**: "Wait 7 days, send at 9:00 AM recipient local."
2. Drag a **Conditional Split**: condition = "SMS Consent equals Subscribed."
   - YES → SMS action.
   - NO → skip directly to next wait step (still send the redemption proximity email later).
3. Configure SMS:
   - Sender ID: RunRec
   - Body: paste exactly (or with the operator-approved screenshot-mechanic update)
   - Quiet Hours: ON (8am–9pm)
   - Smart Sending: OFF
4. Save.

**Reply routing:** A reply with a screenshot is the redemption signal. Eyad/Sal manually mark the profile `bday_gift_redeemed_year = {current_year}` and book the free hour in Skedda. Standard STOP handling.

---

### Touch 03 — Redemption proximity (Email)

- **Channel:** Email
- **Send delay:** Wait 3 days from Touch 02 (DOB +3), send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `When are you redeeming your free hour?`
- **Preview text:** `3 best off-peak windows this month.`
- **Template to use:** Flow / Personal (text-forward, no hero image)
- **Hero image:** None or simple calendar/time-slot card layout. Highlight off-peak windows in accent orange.

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Quick reminder — your free birthday hour is good through <strong>{{ profile.bday_expiry_date }}</strong> ({{ profile.bday_days_remaining }} days from now).</p>
<p>Best off-peak times this month:</p>
<p>→ <strong>Mornings</strong> (10am-12pm) — wide open<br>
→ <strong>Late weekday nights</strong> (10pm-1am) — almost always free<br>
→ <strong>Sunday afternoons</strong> (2-5pm) — quiet</p>
<p>Pick one: <a href="https://therunrec.com/book">therunrec.com/book</a> (screenshot the SMS to redeem)</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — The code is single-use. If you book a longer slot, the first hour is free, the rest at standard rate.</p>
```

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02 (or after the SMS conditional split rejoins), drag a **Time Delay**: "Wait 3 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: condition = "Profile property `bday_gift_redeemed_year` equals `{current_year}`."
   - YES → EXIT.
   - NO → continue to email.
3. Drag an **Email** action. Configure as above. Smart Sending: OFF.
4. Save.

**Inconsistency flag:** The body PS says "The code is single-use" but the SMS in Touch 02 says screenshot-to-redeem (no code). Same operator decision: pick one mechanic. If sticking with screenshot-to-redeem, replace PS with: `If you book a longer slot, the first hour is free, the rest at standard rate.`

The merge tags `{{ profile.bday_expiry_date }}` and `{{ profile.bday_days_remaining }}` need to be calculated. Easiest: at flow entry, set a profile property `bday_expiry_date` = `birthday_dates` + 28 days. This requires a Klaviyo "Update Profile" action at the top of the flow.

---

### Touch 04 — Mid-window deadline (Email)

- **Channel:** Email
- **Send delay:** Wait 11 more days (DOB +14), send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Half your birthday hour is gone.`
- **Preview text:** `Two more weeks. Specific expiry date inside.`
- **Template to use:** Flow / Personal
- **Hero image:** None or hourglass illustration / simple countdown graphic. Minimalist.

**Body content (this is the deployed dashboard copy — flag for the code-mechanic inconsistency):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>You haven't booked your free birthday hour yet. Two more weeks until it expires.</p>
<p>Code: <strong>BDAY-{{ profile.bday_gift_year }}</strong><br>
Expires: <strong>{{ profile.bday_expiry_date }}</strong><br>
Book now: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p>The good slots get taken first.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — If life's been busy and you'd rather give the hour to a friend, just reply with their email. We'll transfer it.</p>
```

**If the operator picks the screenshot-to-redeem mechanic instead** (consistent with Touch 02), replace the body with:

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>You haven't booked your free birthday hour yet. Two more weeks until it expires.</p>
<p>Expires: <strong>{{ profile.bday_expiry_date }}</strong><br>
Screenshot the SMS we sent you on your birthday and email it to us — we'll book your free hour.</p>
<p>The good slots get taken first.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — If life's been busy and you'd rather give the hour to a friend, just reply with their email. We'll transfer it.</p>
```

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay**: "Wait 11 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: same `bday_gift_redeemed_year` check as Touch 03.
   - YES → EXIT.
   - NO → email.
3. Configure email as above. Smart Sending: OFF.
4. Save.

---

### Touch 05 — Final hours (SMS)

- **Channel:** SMS
- **Send delay:** Wait 14 more days (DOB +28), send at 2pm recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
48 hours left on your free birthday hour. Tap to grab a slot: runrec.co/bday

-RunRec
```

(Under 100 characters. Real deadline. Single tap.)

**Klaviyo UI walkthrough — Touch 05:**
1. After Touch 04, drag a **Time Delay**: "Wait 14 days, send at 2:00 PM recipient local."
2. Drag a **Conditional Split**: same redemption check.
   - YES → EXIT.
   - NO → continue.
3. Drag a **Conditional Split**: SMS consent check.
   - YES → SMS.
   - NO → EXIT.
4. Configure SMS:
   - Sender ID: RunRec
   - Body: paste exactly
   - Quiet Hours: ON
   - Smart Sending: OFF
5. Save.

---

## Test plan

The hard part of testing this flow is the date-based trigger. Klaviyo doesn't let you fast-forward dates easily.

**Approach A — temporary date offset (recommended for testing):**
1. Create a test profile with `birthday_dates` set to 8 days from today.
2. Wait 1 day. Touch 01 should fire (because trigger is DOB-7 = today).
3. Klaviyo's flow inspector shows the test profile entered the flow. Use "Send next message immediately" to fire Touches 02, 03, 04, 05 without waiting.
4. Verify each touch's content, timing logic, and conditional splits.

**Approach B — set birthday to today + manual fire:**
1. Set `birthday_dates` to today.
2. Trigger the flow manually via Klaviyo's "Add to flow" admin action.
3. Skip ahead through each touch.

**Things to verify:**
- Touch 01 lands at 10am recipient local timezone, NOT UTC.
- Touch 02 lands at 9am on the actual birthday.
- Touch 03 mentions the correct expiry date (28 days after birthday).
- Touch 04 mentions the correct expiry date.
- Conditional splits exit the flow correctly when `bday_gift_redeemed_year` is set.
- Profile with no SMS consent skips Touches 02, 05 but still receives 01, 03, 04.
- Footer of every email contains business name, address, unsubscribe, manage preferences.

---

## Activation checklist

- [ ] All 5 touches built
- [ ] Profile property `birthday_dates` populated for at least the Birthday Customers list (T8HDuV — currently 19 profiles)
- [ ] Profile property `bday_gift_redeemed_year` exists (numeric)
- [ ] Profile property `bday_expiry_date` is calculated at flow entry via "Update Profile" action
- [ ] Trigger filter ≥1 booking in 12 months is set
- [ ] Conditional splits on Touches 03, 04, 05 all check `bday_gift_redeemed_year` correctly
- [ ] Operator has decided screenshot-to-redeem vs BDAY code mechanic AND copy is consistent across Touches 02, 03, 04, 05
- [ ] Touch 02 and Touch 05 SMS consent splits are wired
- [ ] Quiet Hours ON for both SMS touches
- [ ] Smart Sending OFF on every touch
- [ ] Footer compliance verified
- [ ] Test profile has been through full flow with all conditional paths verified
- [ ] Eyad/Sal know to manually mark `bday_gift_redeemed_year = {current_year}` when a customer redeems

---

## Post-launch monitoring (first 30 days, since the flow is annual)

- **Touch 01 open rate:** Target 50%+ (birthday emails always have high open). Below 35% = subject line problem.
- **Touch 02 SMS delivery rate:** Target 99%+. SMS phone number quality matters here.
- **Redemption rate (% of recipients who redeem within 28 days):** Target 35–50%. This is the headline number. Below 20% means the offer isn't compelling or the redemption mechanic is too friction-heavy.
- **Touch 03 vs Touch 04 vs Touch 05 incremental redemption lift:** each touch should drive 5–10% additional redemption. If Touch 04 (mid-window) drives 0% additional, the body isn't urgent enough.
- **Reply volume to Touch 02 SMS:** This IS the redemption mechanism (screenshot replies). Track these manually.

**Warning signs:**
- Customers complaining the gift is "fake" because they can't figure out how to redeem — copy clarity problem in Touch 02. Rewrite the redemption instruction.
- Spam complaints — birthday emails should never trigger spam complaints if consent is real. Above 0.05% means legacy contacts who don't actually consent.
- The operator forgets to update `bday_gift_redeemed_year` after redeeming → customers receive Touches 03, 04, 05 even after redeeming. Fix: build a Zapier automation or admin checklist for the team.

---

## Common issues for THIS flow

1. **Birthday data is the bottleneck.** Without `birthday_dates` populated for customers, this flow has zero audience. The Birthday Customers list (T8HDuV) only has 19 profiles. The master buildsheet says quarterly manual export from booking history is the path. Run that first import before activating this flow.

2. **The mechanic inconsistency between Touches 02 and 04 is the biggest content issue.** The audit (2026-05-01) flagged this. Operator must pick one mechanic and apply consistently. Recommended: screenshot-to-redeem (per the master buildsheet's Mechanic Doctrine 00.6).

3. **The redemption tracking is manual.** Without an automated booking-system hook, Eyad/Sal must mark `bday_gift_redeemed_year` after each redemption, or the customer will get Touches 03, 04, 05 anyway. Build a simple admin checklist or Zapier flow ("when an inbound email containing 'birthday' arrives at contact@therunrec.com, alert Sal and prompt to mark profile").

4. **Touch 02 SMS at 9am may be too early or too late depending on timezone.** A customer in BC sees their 9am SMS at 6am Pacific. A customer in Halifax sees it at 10am Atlantic. The "9am recipient local" setting handles this — verify that the profile's timezone is set correctly. Most Klaviyo profiles inherit timezone from city/region — if those are blank, Klaviyo defaults to America/Toronto.

5. **Children's birthdays vs adult customers.** The master buildsheet hints at this: a parent customer may have multiple kids on file (`birthday_dates` is an array). This flow as currently built fires per profile, not per child. If Sal wants to send birthday gifts to KIDS at parties (not just to the parent), that's a different flow architecture (likely Flow 05 territory). Confirm intent with operator: is this for the customer's birthday or the customer's kids' birthdays?

6. **The "no offer" PS in Touch 01 ("Don't worry, it's not a 10% off coupon") is a brand-defining moment.** It's also a promise the rest of the flow must keep. If the operator ever decides to swap the free hour for a discount in a future iteration, that PS becomes a lie. Lock the gift-vs-discount decision before launch.

7. **The 28-day window crosses a month boundary for many recipients.** A birthday on the 25th has its window extending into the next month. The merge tag `{{ profile.bday_expiry_date }}` needs to be calculated correctly. Test with a birthday on the 28th to verify month-rollover works.

8. **The operator wakeup campaign before launch.** The master buildsheet flags this: the Birthday Customers list (19 profiles) has had ZERO automation. As soon as Flow 04 launches, those 19 profiles receive their first contact in months — which is fine, but unexpected. **Recommended:** before launching this flow, send a single "we're back" campaign to the Birthday Customers list explaining what's coming. Reduces spam-complaint risk.
