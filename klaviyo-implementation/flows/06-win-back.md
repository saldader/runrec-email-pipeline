# Flow 06 — Win-Back (Lapsed)

**Build time estimate:** ~1.5 hours
**Difficulty:** Medium
**Prerequisites:**
- Pre-flight setup must be complete
- Sender domain `contact@therunrec.com` authenticated
- "Flow / Personal" master template exists
- Profile property `last_booking_date` exists (date) — written by Stripe → Klaviyo on every Successfully Paid event
- "At-Risk" segment exists. Definition: profiles with at least 1 Successfully Paid event AND no booking in last 60+ days. Build this segment first if it doesn't exist.
- The existing live "New - Win Back (Email Flow Part 5)" flow `YvvKXF` (3 emails + 1 SMS over 70 days) should be **paused** before launching this flow. Or, if the operator prefers, extend `YvvKXF` to match this 6-touch spec. This guide assumes building from scratch.
- Decision needed: the dashboard copy uses code mechanics in Touch 02 (`COMEBACK`) and Touch 04 (`RETURN25`) and Touch 05 (`RETURN25`). The master buildsheet's Mechanic Doctrine 00.6 says codes are deprecated — Touches 02, 04, 05 should all use reply-to-redeem. This guide ships the deployed dashboard copy as default but flags the inconsistency.

**Klaviyo glossary** (terms used in this guide):
- **Lapsed customer** — a previous customer who hasn't engaged in 60+ days.
- **Permission to leave** — a final-touch tactic where the email/SMS explicitly invites the customer to unsubscribe. Counter-intuitive but the highest-converting touch in win-back flows.

---

## What this flow does (plain English)

When a previous customer hasn't booked in 60 days, this flow re-engages them with 6 touches over 31 days, each using a different angle:
1. **Curiosity** — "Everything okay over there?" (no offer, just a check-in)
2. **Social offer** — "Bring a friend, both free" (referral framing)
3. **Channel switch** — SMS with specific update ("we added pickleball")
4. **New news** — "What's changed since you left" + 25% off code
5. **Final offer** — "About that thing I mentioned" + real expiry
6. **Permission to leave** — SMS breakup: "If we're not your spot, just reply STOP"

The strategic point: **Hormozi's "ask different questions."** Five "we miss you" emails don't work. Each touch attacks the lapse from a different angle. The breakup at the end (Touch 06) is almost always the highest-converter — counter-intuitively, giving permission to leave makes most people stay.

---

## Trigger — what fires this flow

- **Type:** Date Property Trigger (calculated from `last_booking_date`)
- **Property:** `last_booking_date`
- **Offset:** +60 days
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name: `Flow 06 — Win-Back Lapsed (v2 · 2026-05)`
  3. Trigger type: **Date Property Trigger**
  4. Property: `last_booking_date`
  5. Offset: `60 days after`
  6. Time of day to fire: 10am recipient local

Alternative trigger (if Date Property Trigger isn't available or reliable): **Segment Joined** trigger on the "At-Risk" segment (`60+ days since last booking AND was previously a customer`). Either works.

---

## Filters

- **In `At-Risk` segment** — Klaviyo: "Properties about someone → Segment membership → is in segment → At-Risk."
- **Has not received Win-Back flow in last 6 months** — Klaviyo: "Was in this flow → Zero times in the last 6 months." Prevents over-mailing serial lapsers.
- **Has email marketing consent = subscribed**

---

## Exit conditions

- **New booking placed** — Klaviyo: "Has done event → Successfully Paid → at least 1 time since starting this flow." Profile exits and Flow 02/03 take over.
- **Replies STOP to SMS** — Klaviyo handles automatically.
- **Unsubscribes from email** — Klaviyo handles automatically.

---

## Smart Sending: **OFF** — and why

Win-back flows must complete to clean the list. If Smart Sending skips a touch, the customer doesn't get the breakup email and your list gets dirtier over time. OFF on every touch.

## Quiet Hours: **ON for both SMS touches** (Touches 03 and 06)

8am–9pm recipient local.

---

## Flow map (visual)

```
TRIGGER: 60 days after last_booking_date, 10am local
   ↓
   (Filter: In At-Risk segment, not in flow recently, email consent)
   ↓
Touch 01 EMAIL — Day 60 — Soft "Everything okay over there?" (no offer)
   ↓
Wait 7 days → 10am
   ↓
   Conditional: new booking?
   ├── YES → EXIT
   └── NO → Touch 02 EMAIL — Day 67 — Bring a friend free (COMEBACK code OR reply-to-redeem)
   ↓
Wait 8 days → 2pm
   ↓
   Conditional: new booking?
   ├── YES → EXIT
   └── NO → Touch 03 SMS — Day 75 — Channel switch nudge (pickleball update)
   ↓
Wait 7 days → 10am
   ↓
   Conditional: new booking?
   ├── YES → EXIT
   └── NO → Touch 04 EMAIL — Day 82 — What's new + 25% off (RETURN25 OR reply-to-redeem)
   ↓
Wait 6 days → 10am
   ↓
   Conditional: new booking?
   ├── YES → EXIT
   └── NO → Touch 05 EMAIL — Day 88 — Final offer
   ↓
Wait 3 days → 11am
   ↓
   Conditional: new booking?
   ├── YES → EXIT
   └── NO → Touch 06 SMS — Day 91 — Permission to leave (the breakup)
   ↓
END
```

---

## Touch-by-touch build instructions

### Touch 01 — Soft "anything new?" (Email)

- **Channel:** Email
- **Send delay:** Immediate on trigger (Day 60, 10am local)
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Everything okay over there?`
- **Preview text:** `No offer. Just a check-in.`
- **Template to use:** Flow / Personal (text-forward, no hero image)

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Noticed you haven't been by in 60+ days. Everything okay?</p>
<p>No offer attached to this email. Just a check-in.</p>
<p>A few things changed at RunRec since you were last here — happy to send you the rundown if you reply.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — If you've been busy with work or family, that's the whole point of 24/7. We're still here whenever you need a court.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Email** action right after the trigger.
2. Configure From label, From email, Subject, Preview text exactly as above.
3. Edit content → paste body.
4. Smart Sending: OFF.
5. Save.

---

### Touch 02 — Bring a friend free (Email)

- **Channel:** Email
- **Send delay:** Wait 7 days, send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Bring a friend — first session is on us.`
- **Preview text:** `You + a new face. Both free.`
- **Template to use:** Flow / Personal
- **Hero image:** Two figures playing on Court A — one familiar with the space, one new. Casual, not staged. Avoid stock-photo "high five" cliché.

**Body content (deployed dashboard version with COMEBACK code):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>We get it — life happens. Here's an offer to make coming back easier:</p>
<p>Book any 1-hour court for you and a friend. <strong>We'll waive the cost.</strong></p>
<p>Just one rule: the friend has to be someone who hasn't been to RunRec before.</p>
<p>You both get a free session. We get a new person to introduce to the place. Everyone wins.</p>
<p>Code: <strong>COMEBACK</strong><br>
Book: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Most people who bring a friend end up making it a regular thing. Worth a try.</p>
```

**If operator picks reply-to-redeem (per buildsheet doctrine):** replace the "Code: COMEBACK" + book link with:

```html
<p>Just reply to this email with your ideal day and time, and we'll reserve it for you.</p>
```

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay**: "Wait 7 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: condition = "Has done event → Successfully Paid → 0 times since starting this flow."
   - YES → continue.
   - NO → EXIT.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.
5. Save.

---

### Touch 03 — Personal-feel nudge (SMS)

- **Channel:** SMS
- **Send delay:** Wait 8 days, send at 2:14pm recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Hey {{ first_name|default:'there' }}, it's the team at RunRec. We added pickleball Court A this month + new Saturday morning slots. Worth a look?

-Sal Stop=opt-out
```

(Personalized. Specific update. Casual. Includes opt-out.)

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02, drag a **Time Delay**: "Wait 8 days, send at 2:14 PM recipient local."
2. Drag a **Conditional Split**: booking check.
   - YES → EXIT.
   - NO → continue.
3. Drag a **Conditional Split**: SMS consent check.
4. Configure SMS: Sender RunRec, body as above, Quiet Hours ON, Smart Sending OFF.

---

### Touch 04 — What's new + 25% off (Email)

- **Channel:** Email
- **Send delay:** Wait 7 days, send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `What's changed at RunRec since you left.`
- **Preview text:** `Three concrete things. And a small thank-you for coming back.`
- **Template to use:** Flow / Personal
- **Hero image:** Split image showing one or more of the new features (pickleball lines, audio system close-up, weekend slot calendar). Clean, brand-aligned.

**Body content (deployed dashboard with RETURN25 code):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Three things that are different:</p>
<p><strong>1. Pickleball is now on Court A</strong> (new lines painted, paddles available)<br>
<strong>2. Saturday morning hours are now wide open</strong> (used to be locked for permits)<br>
<strong>3. We added a Bluetooth audio system on every court</strong> (acoustics are unreal — most people don't believe it)</p>
<p>Other than that, same place. Same staff. Same hardwood.</p>
<p>If you want to come check out one of the changes, here's <strong>25% off</strong> on your next single booking: code <strong>RETURN25</strong>, expires in 7 days.</p>
<p>Book: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — The pickleball lines are aggressive. They take some getting used to. But once you do, the game's a riot.</p>
```

**If operator picks reply-to-redeem:** replace the offer paragraph with:

```html
<p>If you want to come check out one of the changes, reply with your preferred day and time and we'll lock it in for you at <strong>25% off</strong>.</p>
```

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay**: "Wait 7 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: booking check. YES → EXIT. NO → continue.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

---

### Touch 05 — The final offer (Email)

- **Channel:** Email
- **Send delay:** Wait 6 days, send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `About that thing I mentioned.`
- **Preview text:** `Final ping. After this the code's gone.`
- **Template to use:** Flow / Personal (text-forward, no hero image)

**Body content (deployed dashboard version):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Just a final ping on the offer.</p>
<p><strong>25% off any single booking. Code RETURN25. Expires {{ profile.return25_expiry|default:'in 7 days' }}.</strong></p>
<p>After that the code's gone.</p>
<p>Book: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — If RunRec just isn't your spot anymore, no hard feelings. Reply STOP and we'll stop emailing. Or just ignore this one and we'll get the message.</p>
```

**If operator picks reply-to-redeem:** replace body with:

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Just a final ping on the offer.</p>
<p>Reply to this email with your preferred day and time to redeem the offer (25% off any single booking).</p>
<p>After {{ profile.return25_expiry|default:'in 7 days' }} the offer's gone.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — If RunRec just isn't your spot anymore, no hard feelings. Reply STOP and we'll stop emailing. Or just ignore this one and we'll get the message.</p>
```

**Klaviyo UI walkthrough — Touch 05:**
1. After Touch 04, drag a **Time Delay**: "Wait 6 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: booking check. YES → EXIT. NO → continue.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

---

### Touch 06 — Permission to leave (SMS — the breakup)

- **Channel:** SMS
- **Send delay:** Wait 3 days, send at 11am recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
If RunRec's not your spot anymore, no hard feelings — just reply STOP and we'll stop emailing. Otherwise, court's yours: runrec.co/back

-RunRec
```

(The most counter-intuitively effective touch in the entire flow. Permission-to-leave converts because people hate being trapped on a list. Some say STOP — that improves your sender reputation. Most stay.)

**Klaviyo UI walkthrough — Touch 06:**
1. After Touch 05, drag a **Time Delay**: "Wait 3 days, send at 11:00 AM recipient local."
2. Drag a **Conditional Split**: booking check. YES → EXIT. NO → continue.
3. Drag a **Conditional Split**: SMS consent check.
4. Configure SMS: Sender RunRec, body as above, Quiet Hours ON, Smart Sending OFF.

**Reply routing:** STOP → automatic unsubscribe. `runrec.co/back` link click → routes to a special booking page with the offer pre-applied. Other replies → live response inbox.

---

## Test plan

1. Create a test profile with `last_booking_date` set to 61 days ago. Add to At-Risk segment manually.
2. Trigger should fire Touch 01 within ~24 hours.
3. Use "Send next message immediately" to fast-forward through Touches 02–06.
4. Verify each touch's content, conditional splits (booking check + SMS consent check), and merge tags.
5. After Touch 06, verify the profile is properly tagged as having completed the flow.

**Edge cases:**
- Make a real booking mid-flow (after Touch 02, before Touch 03). Verify the flow exits at the next conditional check.
- Profile with no SMS consent — should skip Touches 03 and 06 but receive 01, 02, 04, 05.
- Reply STOP to Touch 06 — verify SMS unsubscribe fires correctly.

---

## Activation checklist

- [ ] All 6 touches built
- [ ] At-Risk segment exists and is populated correctly (60+ days since last booking, was previously a customer)
- [ ] Trigger filter "in At-Risk segment, not in flow in last 6 months" set
- [ ] Conditional splits on Touches 02–06 all check for new bookings
- [ ] SMS consent checks on Touches 03 and 06
- [ ] Operator has decided code vs reply-to-redeem doctrine and copy is consistent across Touches 02, 04, 05
- [ ] If using codes, the COMEBACK and RETURN25 codes are configured in the booking system to apply correctly
- [ ] Quiet Hours ON for both SMS touches
- [ ] Smart Sending OFF on every touch
- [ ] Footer compliance on every email
- [ ] Existing `YvvKXF` flow is paused (or operator has decided to extend it instead of building fresh)
- [ ] Test profile has been through full flow end-to-end

---

## Post-launch monitoring (first 30 days)

- **Touch 01 open rate:** Target 30%+. This is a "lapsed" audience; opens will be lower than active customers.
- **Reply volume to Touch 01:** A high reply rate is the strongest signal — those are warm leads. Forward replies to Sal/Izzy for personal follow-up.
- **Touch 02 conversion rate (book a session within 7 days):** Target 5–8%. The bring-a-friend offer is the second-highest converter.
- **Touch 04 conversion rate:** Target 4–7%. Specific updates + 25% off should land for ~5% of recipients.
- **Touch 06 STOP rate:** Target 8–15%. This is GOOD. STOPs at this stage clean the list and improve sender reputation. If STOP rate is below 5%, the breakup language isn't direct enough.
- **Touch 06 booking conversion rate:** Target 8–12%. The breakup paradoxically converts well.
- **Overall flow conversion (book a session within 31 days):** Target 15–25% of all entrants.

**Warning signs:**
- Touch 02 driving complaints ("you said free but charged me") — code mechanic is broken. Verify COMEBACK applies $0 cost, not 100% off + still charges. Common bug.
- Touch 04 RETURN25 code expiring before customer redeems — adjust expiry to 14 days or use rolling expiry.
- High overall unsubscribe rate (>2%) across the flow — the cadence is too aggressive. Consider stretching to 45–60 days total.

---

## Common issues for THIS flow

1. **Code mechanic vs reply-to-redeem inconsistency.** The deployed dashboard uses codes (COMEBACK, RETURN25). The master buildsheet says reply-to-redeem is the standard. Operator must pick one. Recommended: align with buildsheet (reply-to-redeem) — opens real conversations and eliminates code engineering.

2. **The At-Risk segment is the trigger source — make sure it's correctly defined.** Definition: "Has done Successfully Paid event at least once" AND "Has not done Successfully Paid event in the last 60 days." If the segment also includes Newsletter signups who never booked, this flow will spam non-customers. Verify the segment in Klaviyo before going live.

3. **Existing live `YvvKXF` flow conflict.** That flow currently sends 3 emails + 1 SMS over 70 days. If you launch this 6-touch / 31-day flow without pausing `YvvKXF`, lapsed customers will receive 9+ touches over 100 days and complain. Pause `YvvKXF` first.

4. **The "what's new" claims in Touch 03 and Touch 04 must be true.** "We added pickleball." "We added a Bluetooth audio system." These need to be facts. If they're not, the customer comes back and feels lied to. Operator confirms.

5. **The breakup tactic in Touch 06 only works if Touches 01–05 weren't pushy.** Permission-to-leave lands as gracious because the prior touches were varied and value-rich. If the prior touches were 5 "we miss you" emails, the breakup feels manipulative. Maintain the variety.

6. **Friend referral in Touch 02 needs operational support.** "Bring a friend, both free" — when the customer brings a friend, the friend becomes a new lead. Make sure Eyad/Sal capture that friend's email and add them to the Newsletter list (or trigger a Welcome Series for the friend). This is the highest-leverage outcome of the whole flow.

7. **The 31-day window is intentionally tight.** Some operators stretch to 60–90 days. Don't. Lapsed customer attention is short — extending the window dilutes the urgency. Win-back lifecycle research consistently shows 30-day windows outperform longer ones.

8. **The 25% discount in Touches 04/05 trains bad behavior.** A customer who comes back via RETURN25 may expect a discount the next time they lapse. Two mitigations: (a) make the discount one-time and track it via `last_winback_offer_redeemed` property, (b) only fire the discount at Touches 04/05, never earlier. The flow as built does both, but watch for repeat lapsers gaming the system.
