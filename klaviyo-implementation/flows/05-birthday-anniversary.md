# Flow 05 — Birthday Party Anniversary

**Build time estimate:** ~1.5 hours
**Difficulty:** Medium-Hard (per-profile data quality matters)
**Prerequisites:**
- Pre-flight setup must be complete
- Sender domain `contact@therunrec.com` authenticated
- "Flow / Personal" master template exists
- Profile property `last_party_date` exists (date) — populated for the parent customer who booked a previous party
- Profile property `child_name` (string) — for personalization. Optional but high-leverage. If blank, use the fallback patterns documented below.
- Profile property `parent_name` (string) — usually equals `first_name` but may differ for joint accounts.
- Profile property `previous_package` (string) — name of the package they booked last time (HERO / SUPER HERO / SUPER HERO FUN).
- Birthday Customers list `T8HDuV` exists (currently 19 profiles) — most party customers should be in this list.
- Decision needed: the buildsheet says Touch 02 should reference the "Full Facility" (the actual offering); the deployed dashboard still says "Court A." Operator must align before launch.

**Klaviyo glossary** (terms used in this guide):
- **Event-based trigger** — a flow trigger that fires X time after a specific date in a profile property (similar to Flow 04 but with a much longer offset).
- **Branch fallback** — a Klaviyo conditional that picks alternate copy when a merge tag is empty (e.g., if `child_name` is blank, use "your child's").

---

## What this flow does (plain English)

Eleven months after a customer booked a birthday party, this flow sends a memory trigger email ("can you believe it's been a year?"). One week later, a "same date next year, we're holding it for you" email. Two weeks after that, a personal-feel SMS asking "want to chat?" Three weeks later, a package refresh email showing the 3 tiers. Four weeks later (28 days into the flow), a final "should I close the file? Y/N" SMS that forces a decision.

The strategic point: **get there before the parent starts planning.** The window for re-booking parties is narrow — once a parent decides where to have the party, that's it. Triggering 11 months out lets RunRec own the consideration moment before the parent even starts thinking about alternatives.

This is the highest-AOV flow in the system per recipient. Each booked anniversary party is $499–$899. Even a modest 15–20% rebooking rate makes this flow extremely profitable.

---

## Trigger — what fires this flow

- **Type:** Date Property Trigger
- **Property:** `last_party_date`
- **Offset:** +11 months (fires 11 months AFTER the party)
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name: `Flow 05 — Birthday Party Anniversary (v2 · 2026-05)`
  3. Trigger type: **Date Property Trigger**
  4. Property: `last_party_date`
  5. Offset: `11 months after`
  6. Time of day to fire: 10am recipient local

---

## Filters

- **Has booked ≥1 party in last 18 months** — Klaviyo: "Has done event → Successfully Paid → at least 1 time in the last 18 months → where event property `event_type` = `birthday_party`." Filters out non-party bookings. Note: this requires Stripe to tag party bookings with `event_type = birthday_party`. If that property doesn't exist, fall back to: profile is in the Birthday Customers list (`T8HDuV`).
- **Has not received Anniversary flow in last 12 months** — Klaviyo: "Was in this flow → Zero times in the last 12 months."
- **Has email marketing consent = subscribed**

---

## Exit conditions

- **New party booked** — Klaviyo: "Has done event → Successfully Paid → at least 1 time since starting this flow → where event property `event_type` = `birthday_party`." When this fires, profile rolls back to the general party-marketing pool.
- **Day 28 passes** — flow runs out of touches.
- **Unsubscribes** — automatic.

---

## Smart Sending: **OFF** — and why

Annual flow. Single trigger per year. Cannot afford to skip touches. OFF on every touch.

## Quiet Hours: **ON for both SMS touches** (Touches 03 and 05)

8am–9pm recipient local.

---

## Flow map (visual)

```
TRIGGER: Date Property Trigger, last_party_date, +11 months, 10am local
   ↓
   (Filter: ≥1 party in 18mo, not in flow recently, email consent)
   ↓
   Update Profile action: calculate anniversary_date = last_party_date + 12 months
   ↓
Touch 01 EMAIL — 11 months after party — Memory trigger
   ↓
Wait 7 days → 10am
   ↓
   Conditional: new party booked since start?
   ├── YES → EXIT
   └── NO → Touch 02 EMAIL — Same date next year, early lock
   ↓
Wait 7 more days → 11:30am
   ↓
   Conditional: new party booked?
   ├── YES → EXIT
   └── NO → Touch 03 SMS — Personal-feel nudge
   ↓
Wait 7 more days → 10am
   ↓
   Conditional: new party booked?
   ├── YES → EXIT
   └── NO → Touch 04 EMAIL — Package refresh (3 tiers)
   ↓
Wait 7 more days → 10am
   ↓
   Conditional: new party booked?
   ├── YES → EXIT
   └── NO → Touch 05 SMS — Decision-force the close
   ↓
END
```

---

## Touch-by-touch build instructions

### Touch 01 — Memory trigger

- **Channel:** Email
- **Send delay:** Immediate on trigger (11 months after party, 10am local)
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Can you believe it's been a year?`
- **Preview text:** `{{ profile.child_name|default:"your child's" }} last party was 11 months ago.`
- **Template to use:** Flow / Personal
- **Hero image:** Photo from the actual party day if available (with permission). If not, a wide shot of an empty Court A with party setup partially visible. Warm, nostalgic.

**Body content:**

```html
<p>Hey {{ profile.parent_name|default:first_name|default:'there' }},</p>
<p>{{ profile.child_name|default:"Your child's" }} birthday party at RunRec was a year ago this month.</p>
<p>Time flies. Hard to believe it's already been a year since the last one.</p>
<p>We're holding <strong>{{ profile.anniversary_date }}</strong> on Court A in case you want first dibs.</p>
<p>No pressure. Just letting you know it's available.</p>
<p style="font-style:italic;">— Sal &amp; Izzy – RunRec Co-Owners</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Last year's package was the {{ profile.previous_package|default:"HERO" }}. We've added a few things since — happy to walk you through what's new if you reply.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Update Profile** action right after the trigger. Set `anniversary_date` = `last_party_date` + 12 months. (If Klaviyo's date math is limited, calculate this property externally and set it via API at the moment of Successfully Paid for parties.)
2. Drag an **Email** action.
3. Configure From/Subject/Preview as above.
4. Edit content → paste body. Note the merge tag fallbacks: `child_name|default:"Your child's"`, `parent_name|default:first_name|default:'there'`, `previous_package|default:"HERO"`. These are critical — without fallbacks, blank properties render as literal `{{ profile.child_name }}` which looks broken.
5. Smart Sending: OFF.
6. Save.

**Note on doctrine drift:** The body says "Court A." The master buildsheet says the actual offering is "Full Facility" (entire 3-court complex), not Court A alone. Operator must align: change "on Court A" to "at the Full Facility" if going with the corrected offering.

---

### Touch 02 — Same date next year, early lock (Email)

- **Channel:** Email
- **Send delay:** Wait 7 days, send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Same date next year? — we're holding it for you.`
- **Preview text:** `Reply YES to lock {{ profile.anniversary_date }} on Court A.`
- **Template to use:** Flow / Personal (text-forward)

**Body content:**

```html
<p>Hey {{ profile.parent_name|default:first_name|default:'there' }},</p>
<p>Quick follow-up on {{ profile.child_name|default:"your child's" }} anniversary party.</p>
<p><strong>{{ profile.anniversary_date }}</strong> on Court A is yours if you want it. No deposit needed yet — just reply <strong>YES</strong> and we'll lock it in.</p>
<p>After this week we'll open the date back up to the public.</p>
<p style="font-style:italic;">— Sal &amp; Izzy – RunRec Co-Owners</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Saturday afternoons in {{ profile.anniversary_date_month|default:'this month' }} book up fast. Worth grabbing the date now.</p>
```

**Same Court A vs Full Facility note applies.**

The buildsheet says this touch should be: "Booking is for the Full Facility (replaces the older 'Court A' framing — full facility access is the actual offering)" AND the response is "we'll let you know the available slots for you to choose from" (operator-led, not auto-confirmed). If the operator wants to align with the buildsheet, replace the body with:

```html
<p>Hey {{ profile.parent_name|default:first_name|default:'there' }},</p>
<p>Quick follow-up on {{ profile.child_name|default:"your child's" }} anniversary party.</p>
<p><strong>{{ profile.anniversary_date }}</strong> at the Full Facility is yours if you want it. No deposit needed yet — just reply <strong>YES</strong> and we'll let you know the available slots for you to choose from.</p>
<p>After this week we'll open the date back up to the public.</p>
<p style="font-style:italic;">— Sal &amp; Izzy – RunRec Co-Owners</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Saturday afternoons in {{ profile.anniversary_date_month|default:'this month' }} book up fast. Worth grabbing the date now.</p>
```

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay**: "Wait 7 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: condition = "Has done event → Successfully Paid → 0 times since starting this flow."
   - YES (no booking) → continue.
   - NO (booked) → EXIT.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

---

### Touch 03 — Personal-feel nudge (SMS)

- **Channel:** SMS
- **Send delay:** Wait 7 days, send at 11:30am recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Hey {{ profile.parent_name|default:first_name|default:'there' }} — anniversary party season for {{ profile.child_name|default:'your kid' }}. Want to chat? runrec.co/cal

-Sal Stop=opt-out
```

(Personalized. Calendar link `runrec.co/cal` should route to a Calendly or similar booking tool for a 15-minute chat. Signed Sal — feels personal.)

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02, drag a **Time Delay**: "Wait 7 days, send at 11:30 AM recipient local."
2. Drag a **Conditional Split**: same booking check.
   - YES → EXIT.
   - NO → continue.
3. Drag a **Conditional Split**: SMS consent check.
   - YES → SMS.
   - NO → skip to next step.
4. Configure SMS: Sender RunRec, body as above, Quiet Hours ON, Smart Sending OFF.

---

### Touch 04 — Package refresh (Email)

- **Channel:** Email
- **Send delay:** Wait 7 days, send at 10am
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `3 ways to do this year's party.`
- **Preview text:** `Hero · Super Hero · Super Hero Fun. Most parents pick #2.`
- **Template to use:** Flow / Personal (or Birthday Party template if you've built one)
- **Hero image:** Side-by-side cards or photo grid showing the 3 packages. Real photos from past parties (with permission) > stock imagery.

**Body content:**

```html
<p>Hey {{ profile.parent_name|default:first_name|default:'there' }},</p>
<p>If you're still thinking about {{ profile.child_name|default:"your kid's" }} party, here are the three packages:</p>
<p>→ <strong>HERO ($499)</strong> — 2 hours, full court, 6 hoops, basketball/volleyball/dodgeball<br>
→ <strong>SUPER HERO ($699)</strong> — adds pickleball, cornhole, jump rope, hula hoops<br>
→ <strong>SUPER HERO FUN ($899)</strong> — 3 hours, bouncy castle, all of the above</p>
<p>Most parents pick SUPER HERO. The bouncy castle is a hit but optional.</p>
<p>Pick one and we'll get you booked: <a href="https://therunrec.com/birthday-packages">therunrec.com/birthday-packages</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Book by end of week and we'll throw in goodie bags for the kids on the house.</p>
```

**Note on package descriptions:** The deployed dashboard's tier descriptions match the master buildsheet's CORRECTED tier descriptions (per the 2026-05-01 update). If the operator updates pricing or package contents, this body must be updated in lock-step.

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay**: "Wait 7 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: same booking check. YES → EXIT. NO → continue.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

---

### Touch 05 — Decision-force the close (SMS)

- **Channel:** SMS
- **Send delay:** Wait 7 days, send at 10am recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Last call on holding {{ profile.anniversary_date }} for {{ profile.child_name|default:'your kid' }}'s party — should I close the file? Y/N

-Sal
```

(Personalized. Y/N response forces a decision.)

**Klaviyo UI walkthrough — Touch 05:**
1. After Touch 04, drag a **Time Delay**: "Wait 7 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: booking check. YES → EXIT. NO → continue.
3. Drag a **Conditional Split**: SMS consent check.
4. Configure SMS: Sender RunRec, body as above, Quiet Hours ON, Smart Sending OFF.

**Note on subject phrasing:** The buildsheet says this should read "should we close your booking? Yes/No?" (softer "we" + "your booking" + "Yes/No?" phrasing). The deployed dashboard still uses "should I close the file? Y/N" — internal-jargon "the file" + first-person "I." Operator decision: align with buildsheet (recommended) or keep dashboard copy. The buildsheet version is more customer-friendly.

**Reply routing:** Y → escalate to Sal/Izzy to manually book the party. N → mark profile property `anniversary_disposition = declined` and exit. No reply → exit, profile rolls back to general party-marketing pool. STOP → standard.

---

## Test plan

The trigger is 11 months out, so testing requires a temporary date adjustment.

**Approach:**
1. Create a test profile with `last_party_date` set to 11 months ago.
2. Set `child_name = "Aria"`, `parent_name = "Faisal"`, `previous_package = "HERO"`.
3. Trigger fires at 10am local. Touch 01 should arrive same day.
4. Use Klaviyo's "Send next message immediately" to fast-forward through Touches 02–05.
5. Verify each touch's merge tags resolve correctly with both populated values AND with blank values (test the fallback paths).
6. Reply YES to Touch 05 SMS — verify the booking-check conditional fires the EXIT path correctly on the next entry.

**Things to verify:**
- Each merge tag has a working fallback for missing data.
- `anniversary_date` is calculated correctly (last_party_date + 12 months).
- Conditional splits exit the flow if a new booking happens mid-flow.
- Footer compliance on all 3 emails.
- SMS opt-out language present on the first SMS in the flow (Touch 03).

---

## Activation checklist

- [ ] All 5 touches built
- [ ] `last_party_date` populated for at least the Birthday Customers list (T8HDuV)
- [ ] `child_name`, `parent_name`, `previous_package` populated where available; fallbacks tested where blank
- [ ] `anniversary_date` calculated at flow entry via Update Profile action
- [ ] Trigger filter "≥1 party in 18 months" set (or fall back to "in Birthday Customers list")
- [ ] Conditional splits on Touches 02, 03, 04, 05 all check for new bookings
- [ ] Operator has decided Court A vs Full Facility doctrine and copy is consistent
- [ ] Operator has decided "should we close your booking?" vs "should I close the file?" doctrine
- [ ] SMS consent check on Touches 03 and 05
- [ ] Quiet Hours ON for both SMS touches
- [ ] Smart Sending OFF on every touch
- [ ] Footer compliance on every email
- [ ] Test profile has been through full flow with all conditional paths verified

---

## Post-launch monitoring (first 90 days)

Anniversary flows have long latency — give it 90 days before evaluating.

- **Touch 01 open rate:** Target 60%+ (anniversary emails are emotionally weighted; high open rates expected).
- **Touch 02 reply rate (YES):** Target 25%+. This is the early-lock conversion. Below 15% means the body isn't compelling.
- **Touch 04 click-through to package page:** Target 12%+.
- **Touch 05 reply rate (Y/N combined):** Target 30%+. The decision-force should always get a response.
- **Booking conversion rate (% of recipients who book within 28 days):** Target 25–35%. This is the headline number.
- **Average booking value:** Should be $499–$899. Track which package recipients pick most often (the buildsheet says SUPER HERO at $699 is the most popular).

**Warning signs:**
- Touch 01 reply volume with corrections like "her name is Madison, not [first_name]" — your data quality is bad. Run a data audit.
- Customers replying "we already booked" when in fact they did — your booking event isn't firing the EXIT correctly. Check the conditional split.
- High unsubscribe rate (>1%) — the flow is being received as too pushy. Slow down the cadence.

---

## Common issues for THIS flow

1. **Data quality is everything.** This flow is heavily personalized (parent_name, child_name, previous_package, anniversary_date). Bad data = broken-looking emails. Run a quality check on the Birthday Customers list before launch — at minimum, verify every profile has `last_party_date`, `parent_name`, and `child_name`. Profiles with missing `child_name` should be acceptable (fallback to "your child's") but should be flagged for backfill.

2. **The Court A vs Full Facility inconsistency** — the deployed dashboard still says "Court A" in Touches 01 and 02, but the actual party offering is the Full Facility. The buildsheet has the corrected language. Pick one and ship. Recommended: align with buildsheet (Full Facility).

3. **The "should I close the file?" Touch 05 wording** — buildsheet wants "should we close your booking? Yes/No?" Customer-friendlier. Operator decision.

4. **The annual cadence means slow learning.** Unlike Welcome (every signup) or Booking Confirm (every booking), this flow only fires once per year per customer. You won't have meaningful data for 90+ days. Be patient.

5. **The exit condition fires on ANY Successfully Paid event, not just party bookings.** If a parent customer also books a regular court session during the flow window, the EXIT triggers and they don't get the remaining anniversary touches — which means they never get the package refresh email. Fix: scope the exit condition to `event_type = birthday_party` only.

6. **The 11-month trigger means edge cases exist.** A party booked in November 2025 fires this flow in October 2026. If the family moved or the kid aged out, the flow is just noise. Add a soft exit: if the customer has never opened a marketing email in the last 6 months, skip them (consistent with Sunset / Hygiene best practice).

7. **Photos from the original party** are the highest-impact creative asset and the hardest to get. Operator should capture party-day photos with a release form in the future, so Year 2 anniversary emails can include the actual photo. Year 1 launches use empty-court fallback imagery.

8. **The `runrec.co/cal` link in Touch 03** must route to a real calendar booking tool (Calendly, SavvyCal, etc.). If it's broken, this touch is the worst kind of dead-end.
