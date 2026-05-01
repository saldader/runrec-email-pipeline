# Flow 10 — Membership Conversion

**Build time estimate:** 3 hours (4 touches, but the personalized math block is fiddly)
**Difficulty:** Medium (math personalization requires Klaviyo dynamic blocks)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- Profile properties created in Klaviyo (Settings → Profile properties → Add):
  - `booking_count` (numeric) — total Successfully Paid events lifetime
  - `total_spend` (numeric, currency-formatted) — sum of `value` from all Successfully Paid events
  - `membership_tier` (text, values: `Common User`, `Frequent User`, `Super User`, or empty)
  - `membership_status` (text, values: `active`, `cancelled`, `lapsed`, or empty)
- A segment called `Members Active` (we'll create it below).
- A segment called `Conversion Eligible` (we'll create it below).
- A real customer story for Touch 03 with name + photo + permission.
- A landing page at `therunrec.com/membership` showing the 3 tiers (Common $89 / Frequent $139 / Super $359).
- A math breakdown URL (e.g., `therunrec.com/membership/math`) that explains the savings.

### Stripe sub-events caveat

This flow is downstream of Stripe — when someone converts, Flow 11 (Lifecycle) takes over. But Flow 11 needs Stripe `customer.subscription.created` events firing into Klaviyo, which the inventory says aren't currently wired. **Operator action:** verify the Stripe ↔ Klaviyo integration includes subscription events before activating Flow 11. Flow 10 itself only needs `Successfully Paid` events, which are already wired.

---

## What this flow does (plain English)

When a customer hits 3 paid bookings in 90 days (or 2 in 30 days), they've revealed a usage pattern — they're not a one-time visitor anymore. This flow runs them through 4 touches over 14 days: a personalized math reveal showing what they've actually spent vs. what membership would cost, a perks asymmetry email, a real member testimonial, and an SMS first-month-free close.

The flow's job: convert a regular booker into a paying member. Reframing membership as "stop overpaying" instead of "buy a membership."

---

## Trigger — what fires this flow

- **Type:** Metric (`Successfully Paid`)
- **Configuration:** Triggers when someone has 3+ `Successfully Paid` events in last 90 days, OR 2+ in last 30 days.

### How to set this in Klaviyo

Klaviyo's metric trigger doesn't directly support "X times in Y days." Instead, we use a segment that recalculates daily, and trigger on segment join.

**Step 1 — Create the trigger segment:**

In Klaviyo: **Audience → Segments → Create Segment.**

- **Name:** `Conversion Eligible`
- **Definition (multiple OR groups):**

  Group A (3+ in 90 days):
  - Has done `Successfully Paid` **at least 3 times** in last **90 days**

  OR Group B (2+ in 30 days):
  - Has done `Successfully Paid` **at least 2 times** in last **30 days**

  AND (these apply to both groups — outside the OR):
  - Is NOT in segment `Members Active`
  - Is NOT in segment `Conversion Eligible` already (prevents re-trigger loop — actually Klaviyo handles this automatically)

- Click Create.

**Step 2 — Create the Members Active segment (if not done):**

- **Name:** `Members Active`
- **Definition:**
  - Profile property `membership_status` equals `active`
- Create.

**Step 3 — Set the flow trigger:**

- Trigger type: **Segment**
- Segment: `Conversion Eligible`
- Direction: When someone joins this segment

---

## Filters

In the flow, click **Add Filter**:

- **Filter 1:** Profile is in segment `Conversion Eligible` (yes — same as trigger, second-pass safety check)
- **Filter 2:** Profile is NOT in segment `Members Active` (must not already be a member)
- **Filter 3:** Email marketing consent = `subscribed`
- **Filter 4:** Has not received this flow in last 60 days (prevents re-conversion campaigns to people who already declined recently)

---

## Exit conditions

Click **Add Exit Condition**:

- **Exit if:** Profile property `membership_status` becomes `active` (they converted — stop the flow, hand off to Flow 11)
- **Exit if:** Has done `Successfully Paid` 4+ times since flow entry (conversion likely failed — they kept booking single sessions, drop to general nurture)
- **Exit if:** Email marketing consent = `unsubscribed`

---

## Smart Sending: OFF

**Why OFF:** This is a conversion moment. We've earned the right to email them — they've booked 3 times in 90 days. Don't let an unrelated campaign suppress this flow.

## Quiet Hours: 9 AM – 7 PM recipient timezone

Standard hours. No need for weekday-only restriction since these are existing customers.

---

## Flow map (visual)

```
Trigger: joins Conversion Eligible segment
  ↓
Filters
  ↓
Touch 01 (Day 0 immediate) → Email: The math (personalized)
  ↓ Wait 3 days
Touch 02 (Day 3) → Email: What members get that bookers don't
  ↓ Wait 4 days
Touch 03 (Day 7) → Email: Member testimonial + tier comparison
  ↓ Wait 7 days
Touch 04 (Day 14) → SMS: First month waived (the close)
  ↓
EXIT
```

---

## Touch-by-touch build instructions

### Touch 01 — The math (personalized)

- **Channel:** Email
- **Send delay:** Day 0 — fires immediately on segment join. No delay before this touch.
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `Quick math on your last 3 bookings.`
- **Preview text:** `Personalized math. Your actual spend vs. membership cost.`
- **Template:** Flow / Personal

#### Body content (HTML — uses Klaviyo dynamic blocks for math)

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Quick math.</p>

<p>You've booked <strong>{{ person.booking_count|default:"3" }}</strong> sessions at RunRec in the last <strong>90</strong> days.</p>

<p>Total spent: <strong>${{ person.total_spend|default:"267" }}</strong></p>

<p>Here's what club membership would change at your current pace:</p>

<p>→ <strong>5-10% off every booking</strong> (Common User tier saves ~$13.35 on your last 3)<br>
→ <strong>12 hours/year included</strong> (vs paying ~$45-89/hr each time)<br>
→ <strong>Priority booking</strong> — book 6-12 weeks ahead vs 2 weeks for non-members</p>

<p>Being a part of the club isn't mostly about cash savings — it's about getting first dibs on the Saturday peak slots that book up 4 weeks ahead. The math: <a href="https://therunrec.com/membership/math" style="color:#e85a3e;text-decoration:underline;font-weight:600;">therunrec.com/membership/math</a></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Being part of the club isn't for everyone. If you book 1-2x a year, just keep doing what you're doing. But at 3+ bookings every couple months, the math flips.</p>
```

#### How the personalized math actually works

Klaviyo merge tags pull `booking_count` and `total_spend` directly from the profile. These properties need to be UPDATED in Klaviyo each time a `Successfully Paid` event fires. There are two ways to keep them current:

1. **Stripe → Klaviyo native sync** — if the integration is set up to sync custom properties, this is automatic.
2. **Custom Klaviyo Flow that updates properties on every Successfully Paid event** — build a one-step flow: trigger on `Successfully Paid`, action = update profile (`booking_count` += 1, `total_spend` += event.value). This is a small infrastructure flow that should run alongside Flow 10.

**For launch:** verify both properties populate correctly on a test profile. If either is empty, the merge-tag fallback (`|default:"3"`) shows generic numbers — not catastrophic but reduces personalization impact.

#### Klaviyo UI walkthrough for Touch 01

1. From the empty flow canvas (after trigger setup), click **"+"** → **Email**.
2. Name: `Flow 10 — T01 — The Math`.
3. **Set delay BEFORE this email to `0 seconds`** — fires immediately on segment join.
4. From name + email + reply-to as above.
5. Subject + preview text.
6. Edit content → Flow / Personal template.
7. Drag a Text block, switch to HTML mode, paste body content.
8. **Verify merge tags render correctly:** click "Preview" → pick a real customer profile from your account. Confirm `{{ person.booking_count }}` and `{{ person.total_spend }}` show real numbers, not the `|default` fallbacks.
9. Smart Sending: OFF.

---

### Touch 02 — What members get that bookers don't (yet)

- **Channel:** Email
- **Send delay:** Wait `3 days` after Touch 01 → Day 3 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `What members get that you don't (yet).`
- **Preview text:** `5 perks. The first one is the real value.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Five things members get that single-booking customers don't:</p>

<p><strong>1. Book 6-12 weeks ahead</strong> (regulars book 2 weeks ahead)<br>
<strong>2. 5-10% off every booking</strong> (auto-applied at checkout, no code)<br>
<strong>3. 10% off products</strong> (RunRec Ball, gear, accessories)<br>
<strong>4. Annual member-only party</strong><br>
<strong>5. Priority access</strong> to new programs and events</p>

<p>The math we showed yesterday is just the cash savings. <strong>The real value is the priority booking</strong> — Saturday peak slots get taken 4 weeks out.</p>

<p>Tiers: <a href="https://therunrec.com/membership" style="color:#e85a3e;text-decoration:underline;font-weight:600;">therunrec.com/membership</a></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, The Super User tier ($359/mo) gets 48 hours/year + book 12 weeks ahead + member party + Swag Kit. The real value isn't the hours — it's never being shut out of a Saturday peak slot.</p>
```

**Cross-flow consistency note:** "regulars book 2 weeks ahead" must match Flow 11 T01 + T02. The buildsheet flagged this — single source of truth: members book 6-12 weeks, regulars book 2 weeks. Don't deviate.

---

### Touch 03 — Member testimonial + tier comparison

- **Channel:** Email
- **Send delay:** Wait `4 days` after Touch 02 → Day 7 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `How {{name}} did the math and switched.`
- **Preview text:** `A real member who started exactly where you are.`
- **Template:** Flow / Personal

**About `{{name}}`:** Hard-code with a real member's first name (e.g., "Marcus") for launch. Get permission. Replace every `{{name}}` below with the actual name.

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Meet {{name}}. Started as a regular booker like you.</p>

<p>Booked 3-4x a month for 6 months. Spent ~$1,200.</p>

<p>Switched to a Frequent User membership at the 6-month mark.</p>

<p><strong>The math:</strong> $1,200/year on bookings is roughly $100/month spent on courts. Frequent User membership ($139/mo) costs slightly more on paper. But — Frequent gets 18 hours/year priority-booked (Saturday peak slots locked in 8 weeks ahead), plus the member party.</p>

<p><strong>{{name}}'s take: "I stopped competing for Saturday slots. That alone was worth it. I also book 2-3 more times a month because it feels paid for."</strong></p>

<p>Tier comparison: <a href="https://therunrec.com/membership" style="color:#e85a3e;text-decoration:underline;font-weight:600;">therunrec.com/membership</a></p>

<p style="font-style:italic;margin-top:20px;">Izzy, RunRec</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, {{name}} is a real person. If you reply YES we'll connect you — he's happy to talk about the switch.</p>
```

**Image:** Real photo of the member on the court (with permission). If no photo, use a 3-tier comparison card graphic instead.

#### About the "reply YES, connect you" PS

This requires team triage. Set up Gmail filter on `contact@therunrec.com`: any inbound containing "YES" within 7 days of Touch 03 → label `Member Connect Request`. Team forwards to {{name}} with intro.

---

### Touch 04 — First month waived (SMS, the close)

- **Channel:** SMS
- **Send delay:** Wait `7 days` after Touch 03 → Day 14 total
- **Sender label:** `RunRec` (NOT a person — final close goes to brand voice)
- **Send time:** 9 AM – 12 PM recipient timezone (morning send, "this week" deadline framing)

#### Body content (SMS)

```
Reply YES this week and we'll waive your first month. First month is on us — just say the word. -RunRec Team · Stop=opt-out
```

#### How the redemption works

The customer texts back `YES` (or any positive reply). The team:
1. Receives the SMS reply in Klaviyo's SMS inbox.
2. Manually creates the Stripe subscription with first month at $0.
3. Tags the profile `membership_status = active`, `membership_started_date = today`.
4. Profile exits Flow 10, triggers Flow 11 (Lifecycle) via the Stripe `subscription.created` event.

**Operator action:** Document the team SOP for handling SMS replies. Who monitors the Klaviyo SMS inbox? What's the SLA (target: 2 hours)? Where does the Stripe creation happen?

#### Klaviyo UI walkthrough for Touch 04

1. Drag Time Delay → 7 days.
2. Click **"+"** → SMS.
3. Name: `Flow 10 — T04 — First Month Waived`.
4. Sender Profile: `RunRec` (default brand sender).
5. Body content from above.
6. Send time window: 9 AM – 12 PM recipient timezone.
7. Smart Sending: OFF.
8. Filter: `Properties → SMS Marketing == subscribed`. If false, skip (flow ends without SMS — that's OK, but log this on the profile so the team can email-redeem manually).

---

## Test plan

Before activating:

1. **Create a test profile** with these properties manually set:
   - `booking_count = 3`
   - `total_spend = 267`
   - email = your email, phone = your phone
2. **Manually add to the Conversion Eligible segment** (or wait for the segment to recalculate after firing 3 fake `Successfully Paid` events on the test profile).
3. Verify the profile enters the flow within 1 hour.
4. Touch 01 should arrive within minutes (Day 0 immediate).
5. Open the email — verify `{{ person.booking_count }}` shows `3` and `{{ person.total_spend }}` shows `$267`.
6. Test all 4 touches via Klaviyo's preview.
7. **Test the exit condition:** halfway through the flow, manually set `membership_status = active` on the test profile. Verify the profile exits within 24 hours.
8. **Test the SMS:** receive Touch 04 on your real phone. Reply YES. Verify the reply lands in Klaviyo's SMS inbox.
9. **Test the math accuracy:** create a test profile with `booking_count = 5` and `total_spend = 445`. Verify the email shows those exact numbers, not the defaults.

---

## Activation checklist

- [ ] All 4 touches built
- [ ] Trigger segment `Conversion Eligible` populated and updating daily
- [ ] `Members Active` segment exists
- [ ] Profile properties `booking_count`, `total_spend`, `membership_status` all populated on existing customer profiles (test against 5 random customer profiles — if any have empty values, the math personalization will fall back to defaults)
- [ ] Property update mechanism in place (Stripe sync OR helper flow that updates props on Successfully Paid)
- [ ] Membership landing page live at `therunrec.com/membership`
- [ ] Math breakdown page live at `therunrec.com/membership/math`
- [ ] Real customer story (name, quote, photo, permission) for Touch 03
- [ ] Smart Sending OFF on all touches
- [ ] Inbox triage SOP for "reply YES" responses (Touch 03 + Touch 04)
- [ ] Manual Stripe-subscription-creation SOP for SMS YES replies
- [ ] Operator approval

---

## Post-launch monitoring (first 14 days)

- **Trigger volume.** How many profiles entered the flow? Should match the rate of "3+ bookings in 90 days" customers — typically 5-15% of monthly active bookers.
- **Touch 01 open rate.** Target 55%+ — these are warm, recently-engaged customers. Below 35% = subject line issue.
- **Touch 04 SMS reply rate.** Target 5%+ saying YES. This is the core conversion metric.
- **Conversion rate (entered flow → became member).** Target 8-15%. Below 5% means the offer or messaging needs rework.
- **Time-to-conversion.** Members who convert during Flow 10 — are they converting on Touch 01 (math reveal), or holding out until Touch 04 (SMS close)? Tells you which touch is doing the work.
- **Exit at "4th booking" condition.** People who keep booking single sessions but never convert. After 60 days, consider building a "frequent booker, never-convert" win-back campaign for this cohort.

---

## Common issues for THIS flow

**Issue: Math shows generic defaults, not real numbers.**
- `booking_count` and `total_spend` aren't populated on the profile. Check Stripe sync or build the helper flow to populate them.
- Verify the merge tag syntax: `{{ person.booking_count }}` not `{{booking_count}}`.

**Issue: Profile receives Flow 10 even though they're already a member.**
- The `Members Active` segment isn't catching them. Check `membership_status` — it might say `Active` (capitalized) instead of `active`. Standardize.
- Or: the flow filter "Not in Members Active" wasn't added. Re-check.

**Issue: Customer hits 3 bookings, but flow doesn't fire.**
- Segment recalculation is slow (Klaviyo runs segment refreshes every few hours, not real-time for complex segments). Wait 4-6 hours.
- If still not firing, the metric source might not be `Successfully Paid` (Stripe). Verify which integration is firing the event.

**Issue: SMS reply YES, but no one books them.**
- Inbox triage SOP is broken. Re-establish: who monitors? What's the SLA?

**Issue: Profile re-enters flow 30 days later.**
- 60-day re-entry filter is wrong. Re-check filter setup.

---

## Template note

All emails use **Flow / Personal** template. Footer required:
- `RunRec` · `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8` · unsubscribe link · manage preferences link
