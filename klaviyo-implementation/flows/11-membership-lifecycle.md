# Flow 11 — Membership Lifecycle

**Build time estimate:** 8 hours (longest flow — 365-day duration with 6 main + 2 branch touches, requires Stripe subscription event integration)
**Difficulty:** Hard (Stripe subscription events not currently wired; conditional split logic for cancellation branch)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- **Stripe → Klaviyo subscription events firing.** Currently the inventory shows only `Successfully Paid` (one-time payment) events. This flow needs:
  - `Subscription Created` (Stripe `customer.subscription.created`) — triggers the flow
  - `Subscription Cancelled` (Stripe `customer.subscription.deleted`) — triggers the cancel branch (C1, C2)
  - **Operator action:** verify Stripe ↔ Klaviyo integration scope. The default Stripe-Klaviyo connector may not include subscription events. May need to extend via Stripe webhooks → custom Klaviyo metric.
- Profile properties (Settings → Profile properties → Add):
  - `membership_tier` (text — `Common User` / `Frequent User` / `Super User`)
  - `membership_started_date` (date)
  - `membership_status` (text — `active` / `cancelled` / `lapsed`)
  - `last_membership_renewal` (date)
  - `bookings_count_membership_year` (numeric — bookings since last renewal, for year-in-review math)
  - `hours_played_membership_year` (numeric)
  - `savings_amount` (numeric — calculated, total saved vs single-booking pricing)
- A 3-question survey link for Touch 03 (Typeform/Google Forms — get URL).
- A `B2B Win-back` or rejoin URL for branch C2.
- A landing page with all 3 tiers and upgrade/downgrade options.
- The math from the buildsheet: Common $89/mo · Frequent $139/mo · Super $359/mo. Pull live data into the year-in-review (T04).

### Critical: the cancel branch math

Branch C2 originally said "first month back at 50% off." The buildsheet shows the source HTML reads `[discount removed by Sal]`. Operator removed the 50% offer pre-launch. Decide before launching:
- Option A: Remove the discount entirely from C2 (just "door's open" message, no offer).
- Option B: Restore the 50% offer.
- Option C: Different incentive (e.g., "first month at $0" — matches Flow 10 close).

**Default to Option A** for launch — pure goodwill, no transactional sweetener at the goodbye moment. Operator can layer in C2 v2 later if comeback rates are low.

---

## What this flow does (plain English)

When someone starts a membership (Stripe subscription created), this 365-day flow runs through the entire membership year: welcome, week-1 activation nudge, 30-day pulse check, 60-day-before-renewal year-in-review, 30-day-before renewal options, and 7-day-before SMS. If they cancel at any point, a 2-touch branch fires (graceful goodbye + come-back invitation).

The flow's job: prevent silent churn. Most cancellations happen 30-60 days before renewal when members realize they haven't used the membership enough. The year-in-review (T04) shows them they DID use it.

---

## Trigger — what fires this flow

- **Type:** Metric (custom event)
- **Metric:** `Subscription Created` (or whatever Stripe-Klaviyo names the metric — could be `Stripe Subscription Created`)
- **Configuration:** Trigger fires when this metric event is received for a profile

### How to verify the metric exists

In Klaviyo: **Analytics → Metrics → search "subscription"**.

If you see `Subscription Created` or similar, you're good. If not, the Stripe integration isn't sending subscription events. Stop here, fix the integration, then continue.

To extend Stripe-Klaviyo with subscription events: in Stripe Dashboard → Webhooks → Add endpoint pointing to Klaviyo's webhook URL (or use Zapier as a bridge). Subscribe to `customer.subscription.created` and `customer.subscription.deleted`. This is dev work — coordinate with Eyad/Rob.

---

## Filters

In flow trigger configuration:

- **Filter 1:** New member — `membership_started_date` is less than 7 days ago (so a profile with an old subscription doesn't accidentally trigger this on integration sync).
- **Filter 2:** Email marketing consent = `subscribed`.

---

## Exit conditions

- **Exit if:** `membership_status` becomes `cancelled` AND profile reaches branch C2 → exit after C2.
- **Exit if:** `membership_status` is `lapsed` (payment failure, multiple retries failed).
- **Exit if:** Email marketing consent = `unsubscribed`.

NOTE: cancellation does NOT exit the flow immediately — it routes to the cancel branch (C1 + C2), then exits.

---

## Smart Sending: OFF

**Why OFF:** These are paying members. They've earned the dedicated send. Don't let other campaigns suppress lifecycle-critical emails (especially the renewal sequence at Days 305, 335, 358).

## Quiet Hours: 9 AM – 7 PM recipient timezone

Standard hours. No weekday restriction.

---

## Flow map (visual)

```
Trigger: Subscription Created event
  ↓
Filters
  ↓
Touch 01 (Day 0 immediate) → Email: Welcome to the family
  ↓ Wait 7 days
Touch 02 (Day 7) → Email: Don't sit on those hours
  ↓ Wait 23 days
Touch 03 (Day 30) → Email: First-month pulse (survey)
  ↓ Wait 275 days
Touch 04 (Day 305 / 60 days before renewal) → Email: Year in numbers
  ↓ Wait 30 days
Touch 05 (Day 335 / 30 days before renewal) → Email: Renewal options
  ↓ Wait 23 days
Touch 06 (Day 358 / 7 days before renewal) → SMS: Quick yes/no
  ↓
EXIT (renewal happens or doesn't)

CANCEL BRANCH (fires any time membership_status changes to cancelled):
  ↓
C1 (Cancel +0) → Email: Door's always open
  ↓ Wait 7 days
C2 (Cancel +7) → Email: Come back invitation
  ↓
EXIT
```

---

## Cancel branch architecture in Klaviyo

Klaviyo doesn't have a native "cancel branch from any point in flow." There are two ways to wire this:

### Option A — Single second flow that listens for cancellation

Build the cancel branch as a SEPARATE Klaviyo flow:
- **Flow name:** `Flow 11b — Membership Cancel Branch`
- **Trigger:** Custom event `Subscription Cancelled`
- **Filter:** Profile was a member (`membership_started_date` exists)
- **Touches:** C1 immediate, C2 at +7 days
- **Exit condition:** `Subscription Created` fires again (they came back, exit cleanly)

This is the cleaner architecture — easier to maintain, easier to debug, easier to swap C2 messaging.

### Option B — Single flow with conditional split inside

Inside Flow 11, after each main-sequence touch, drop in a Conditional Split:
- "Profile property `membership_status` equals `cancelled`?"
  - YES → route to C1 → wait 7 days → C2 → EXIT
  - NO → continue to next main touch

This puts everything in one flow but creates 6 conditional splits in the canvas (one after each main touch). Hard to debug.

### Recommendation: **Option A — separate flow.** Build Flow 11 (main sequence) and Flow 11b (cancel branch) as two distinct flows. Document them as a pair.

The build instructions below assume Option A.

---

## Touch-by-touch build instructions (MAIN sequence)

### Touch 01 — Welcome to the family

- **Channel:** Email
- **Send delay:** Day 0 — fires immediately on Subscription Created event
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `Welcome to the family — here's what's yours.`
- **Preview text:** `Tier, hours, booking window. Auto-applied at checkout.`
- **Template:** Flow / Personal (premium variant if you have one — buildsheet calls for VIP/Status template; default to Flow/Personal if VIP/Status isn't built yet)

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>You're in.</p>

<p><strong>Your tier:</strong> {{ person.membership_tier|default:"Common User" }}<br>
<strong>Member since:</strong> {{ person.membership_started_date|date:"M j, Y" }}<br>
<strong>Hours included:</strong> {{ person.hours_included|default:"12" }} per year<br>
<strong>Booking window:</strong> {{ person.booking_window|default:"6" }} weeks ahead (vs 2 weeks for regular)</p>

<p>Your first booking is yours to make whenever you want. The system already knows you're a member — discounts and priority access are auto-applied at checkout.</p>

<p>Book: <a href="https://therunrec.com/book" style="color:#e85a3e;text-decoration:underline;font-weight:600;">therunrec.com/book</a></p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Booking your first member session in week 1 is the single move that makes membership feel like yours. Don't let it sit.</p>
```

**Hours included by tier (hard-code if you don't have the property populated):**
- Common User: 12 hours
- Frequent User: 18 hours
- Super User: 48 hours

**Booking window by tier:**
- Common User: 6 weeks
- Frequent User: 8 weeks
- Super User: 12 weeks

If `hours_included` and `booking_window` aren't profile properties yet, you can either: (a) create them and populate per-member, OR (b) hard-code per-tier using a Klaviyo conditional block inside the email body. For launch, option (a) is cleaner.

---

### Touch 02 — Don't sit on those hours (activation)

- **Channel:** Email
- **Send delay:** Wait `7 days` after Touch 01 → Day 7 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Don't sit on those member hours.`
- **Preview text:** `3 underused perks. Most useful first move inside.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Quick reminder — you're a member now. Some of the most underused perks:</p>

<p>→ <strong>Saturday/Sunday peak slots</strong> are bookable 6-12 weeks out (vs 2 weeks for non-members). Book a recurring slot.<br>
→ <strong>The 5-10% booking discount</strong> auto-applies. No code needed.<br>
→ <strong>The 10% product discount</strong> works on balls, gear, and RunRec Ball ($49.99 minus 10% = $44.99).<br>
→ <strong>Member-only events</strong> (next one: {{next_event}}).</p>

<p>Most useful first move: <strong>book a Saturday morning slot 4 weeks out.</strong> Saturday 10am-12pm Court A is the most-requested member slot.</p>

<p>Book: <a href="https://therunrec.com/book" style="color:#e85a3e;text-decoration:underline;font-weight:600;">therunrec.com/book</a></p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you don't use the hours, they don't roll over. Use 'em.</p>
```

`{{next_event}}` is a hard-code — pick the actual next member event (e.g., "Annual member party, Aug 15"). Update quarterly.

**Cross-flow consistency:** "vs 2 weeks for non-members" must match Flow 10 T02. Same fact, same wording.

---

### Touch 03 — First-month pulse (survey)

- **Channel:** Email
- **Send delay:** Wait `23 days` after Touch 02 → Day 30 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `How's your first month going?`
- **Preview text:** `3-question survey. 2 minutes.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>30-day check-in.</p>

<p>Quick survey — 3 questions, 2 minutes:</p>

<p>→ How's the booking experience?<br>
→ Anything missing or confusing?<br>
→ Anything we could fix?</p>

<p>Survey: <a href="{{survey_url}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">{{survey_url}}</a></p>

<p>We read every response. <strong>If anything's off, we want to know now.</strong></p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, We read every reply. The members who tell us what's working tend to be the members who stick around.</p>
```

Replace `{{survey_url}}` with the actual Typeform/Google Forms URL.

**Operator action:** check survey responses weekly. Surface findings to the team.

---

### Touch 04 — Year in numbers (60 days before renewal)

- **Channel:** Email
- **Send delay:** Wait `275 days` after Touch 03 → Day 305 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Your year at RunRec, in numbers.`
- **Preview text:** `Hours played, bookings made, money saved.`
- **Template:** Flow / Personal (or VIP/Status if available — this email deserves the premium treatment)

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Your member year wraps up in 60 days. Here's the recap:</p>

<p>→ Bookings made: <strong>{{ person.bookings_count_membership_year|default:"24" }}</strong><br>
→ Hours played: <strong>{{ person.hours_played_membership_year|default:"36" }}</strong><br>
→ Sports tried: <strong>{{ person.sports_count_membership_year|default:"3" }}</strong><br>
→ Money saved vs. single bookings: <strong>${{ person.savings_amount|default:"328" }}</strong></p>

<p><strong>The math:</strong> at single-booking rates, this year would have cost you <strong>${{ person.would_have_cost|default:"1,396" }}</strong>. Membership cost: <strong>${{ person.membership_cost_year|default:"1,068" }}</strong>. Net savings: <strong>${{ person.savings_amount|default:"328" }}</strong>.</p>

<p>You're already on the math right side of the line.</p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Renewal happens automatically on {{ person.renewal_date|date:"M j" }}. We'll email you a few times before then so you can change tiers if you want to up- or downgrade.</p>
```

#### How to populate the year-in-review math

You need a helper flow that calculates these properties at Day 300:
- `bookings_count_membership_year` — count of `Successfully Paid` events since `membership_started_date`
- `hours_played_membership_year` — sum of session-length values
- `sports_count_membership_year` — distinct count of `last_sport_booked` values
- `savings_amount` — manual calc: (sum of single-booking-rate-equivalents) − (membership cost paid)
- `would_have_cost` — sum of single-booking rates if no membership
- `membership_cost_year` — straightforward: monthly fee × 12
- `renewal_date` — set when membership starts; updates on renewal

Build a small helper flow: trigger on "300 days after `membership_started_date`," action = update profile properties from Klaviyo metric aggregations.

If this is too complex for launch, ship Touch 04 with hard-coded "average member" stats and personalize over time.

---

### Touch 05 — Renewal options (30 days before renewal)

- **Channel:** Email
- **Send delay:** Wait `30 days` after Touch 04 → Day 335 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Your renewal's coming up — anything you want to change?`
- **Preview text:** `Stay, upgrade, or downgrade. Three options.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Your membership renews in 30 days, on <strong>{{ person.renewal_date|date:"M j, Y" }}</strong>.</p>

<p>Three options:</p>

<p><strong>1. Stay on {{ person.membership_tier|default:"Common User" }}.</strong> Same rate, no action needed.<br>
<strong>2. Upgrade to {{ person.next_tier|default:"Frequent User" }}.</strong> More hours, more flexibility. ${{ person.upgrade_cost|default:"139" }}/month.<br>
<strong>3. Downgrade to {{ person.lower_tier|default:"Common User" }}.</strong> Less hours, less commitment. ${{ person.downgrade_cost|default:"89" }}/month.</p>

<p>Reply with the tier you want or just stay put.</p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, A few new things we're rolling out for members in {{next_quarter}}: {{upcoming_perks}}. Worth knowing before you decide.</p>
```

#### Tier-aware logic

`{{ person.next_tier }}` and `{{ person.lower_tier }}` need calculation logic:
- If current = Common: next = Frequent ($139), lower = none (cancel only)
- If current = Frequent: next = Super ($359), lower = Common ($89)
- If current = Super: next = none (top tier), lower = Frequent ($139)

This is a 3-way conditional. Easiest approach: build 3 versions of Touch 05 in Klaviyo (one per tier), and use a Conditional Split BEFORE Touch 05 to route by `membership_tier`.

OR: hard-code one variant for launch (assume Common User → upgrade to Frequent), accept the imprecision for non-Common members for v1, fix in v2.

`{{next_quarter}}` and `{{upcoming_perks}}` are manual hard-codes — update quarterly.

**Operator action:** monitor "reply with tier" responses. Stripe subscription tier changes need to be processed manually unless self-serve membership management exists.

---

### Touch 06 — Quick yes/no (SMS, 7 days before renewal)

- **Channel:** SMS
- **Send delay:** Wait `23 days` after Touch 05 → Day 358 total
- **Sender label:** `RunRec`
- **Send time:** 1 PM – 4 PM recipient timezone (afternoon, "this week" framing)

#### Body content (SMS)

```
Membership renews in 7 days. Reply OK to stay or any other tier name to switch. -RunRec Team · Stop=opt-out
```

#### Klaviyo UI walkthrough for Touch 06

1. Drag Time Delay → 23 days.
2. Click **"+"** → SMS.
3. Sender Profile: `RunRec`.
4. Body content from above.
5. Send time: 1-4 PM recipient timezone.
6. Smart Sending: OFF.
7. Filter: SMS Marketing subscribed. (Most members signed up via Stripe, may not have SMS consent — flow continues without SMS in that case.)

After Touch 06, the flow has no more touches in the main sequence. Stripe will auto-renew (or auto-cancel if payment fails) at Day 365. The flow ends here.

---

## Touch-by-touch build instructions (CANCEL BRANCH)

These belong in the SEPARATE flow `Flow 11b — Membership Cancel Branch`.

### Branch C1 — Door's always open

- **Trigger (separate flow):** Custom event `Subscription Cancelled`
- **Channel:** Email
- **Send delay:** Day 0 — fires immediately on cancellation
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Thanks for being a member — door's always open.`
- **Preview text:** `No questions, no guilt trip. Life happens.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>Got the cancellation. <strong>No questions, no guilt trip — life happens.</strong></p>

<p>A few things since you joined: {{recent_changes}}</p>

<p>The door's always open if you want to come back. <strong>No contract, no rejoining fee.</strong></p>

<p style="font-style:italic;margin-top:20px;">Sal &amp; Izzy — RunRec Co-Owners</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you'd be willing to share why you're leaving, just reply. Helps us not make the same mistake twice.</p>
```

`{{recent_changes}}` is a hard-code — update quarterly with what's been added at the facility (e.g., "We added pickleball courts and Saturday extended hours since you joined.").

**Sig note:** This is the ONE email in the system signed by both Sal AND Izzy (per the buildsheet — "founder warmth at the goodbye moment matters more than at any other touch"). Don't shorten to just "RunRec Team."

---

### Branch C2 — Come back invitation

- **Channel:** Email
- **Send delay:** Wait `7 days` after C1 → Cancel +7
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `An option, if you want it.`
- **Preview text:** `30-day window. No-strings re-entry.`
- **Template:** Flow / Personal

#### Body content (Option A — discount removed per Sal)

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>A week since you cancelled. If you've reconsidered, here's the deal:</p>

<p><strong>Rejoin within 30 days, no rejoin fee, no waiting period.</strong> Same tier, same rate, same perks — picks up right where you left off.</p>

<p><a href="{{rejoin_url}}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">{{rejoin_url}}</a></p>

<p>After 30 days, you'd start fresh on whatever tier is current.</p>

<p style="font-style:italic;margin-top:20px;">— Sal, RunRec Co-Owner</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, Most cancelled members realize they used the membership more than they thought. Door's open whenever.</p>
```

Replace `{{rejoin_url}}` with the actual rejoin link (`therunrec.com/membership/rejoin`).

**If operator restores the discount (Option B):** add `Your first month back is 50% off.` after the "no waiting period" sentence and update PS accordingly.

**Sig note:** Signed Sal alone (not Sal & Izzy) — the buildsheet calls for "personal close to a personal gesture."

---

## Test plan

Before activating:

1. **Test the trigger:** fire a fake `Subscription Created` event for your test profile (use Klaviyo's API or have dev fire one). Verify Flow 11 starts.
2. **Test Touch 01 immediate:** verify it arrives within 5 minutes of the event.
3. **Test merge tags:** set `membership_tier = "Frequent User"`, `membership_started_date = today` on the test profile. Verify Touch 01 shows correct tier and date.
4. **For Touches 02-06, manually advance the flow:** Klaviyo lets you "skip wait" on individual profiles for testing. Use this to verify each touch.
5. **Test the cancel branch:** fire a `Subscription Cancelled` event for the test profile. Verify Flow 11b starts AND main Flow 11 exits cleanly.
6. **Verify NO double-fires:** the test profile should not receive Touch 02 from main flow if they cancelled before Day 7.
7. **Test SMS Touch 06:** receive the SMS on real phone. Reply "OK" — verify reply lands in Klaviyo SMS inbox.

---

## Activation checklist

- [ ] Stripe subscription events firing into Klaviyo (verified in Metrics list)
- [ ] All 6 main + 2 branch touches built (across two separate flows)
- [ ] Profile properties created and populated for first cohort of members
- [ ] Year-in-review helper flow built (or hard-coded fallback approved)
- [ ] Survey URL is real and live (Touch 03)
- [ ] Rejoin URL is real and live (C2)
- [ ] Cancel branch decision documented (Option A no-discount vs Option B with-discount)
- [ ] Tier-aware logic for Touch 05 chosen (3 conditional variants vs 1 hard-coded)
- [ ] Smart Sending OFF on all touches
- [ ] Sender Profile `RunRec` exists for SMS
- [ ] Tested: subscription created → Flow 11 starts; subscription cancelled → Flow 11b starts AND Flow 11 exits
- [ ] Operator approval — note this flow runs for 365 days, mistakes compound

---

## Post-launch monitoring

Daily for first 14 days, then weekly:

- **New member entries to flow.** Match against Stripe new subscription count.
- **Touch 01 open rate.** Target 70%+ — these are paying customers, opening welcome.
- **Touch 03 (Day 30) survey responses.** Target 15%+ response rate. Reading every one is mandatory — surface to team.
- **Cancel branch entries.** Match against Stripe cancellation count. Mismatch = integration issue.
- **C2 rejoin rate.** Target 8-12% return within 30 days. Below 5% = the cancel branch isn't compelling.
- **Touch 04 (Year in numbers) at Day 305 — open rate.** This is the biggest retention lever in the flow. Target 60%+ open. Below 40% = subject line problem (test variants like "Your year at RunRec." or "$[X] saved this year.").
- **Touch 05 reply distribution.** What % stay on current tier vs upgrade vs downgrade? Healthy: 80% stay, 10% upgrade, 5% downgrade, 5% cancel.

---

## Common issues for THIS flow

**Issue: Trigger doesn't fire when subscription created.**
- Stripe-Klaviyo integration doesn't include subscription events. Check Stripe Dashboard → Webhooks → verify subscription events are sent to Klaviyo.
- Or: metric name in Klaviyo doesn't match what trigger expects. Look up exact metric name in Analytics → Metrics.

**Issue: Cancel branch doesn't fire.**
- `Subscription Cancelled` metric not firing. Same Stripe webhook issue.
- Flow 11b filter is too restrictive. Check filters.

**Issue: Member receives Touch 02 (Day 7) AFTER they already cancelled.**
- Main flow exit condition is missing or broken. Re-check exit condition: "membership_status == cancelled."
- Klaviyo flow exit conditions evaluate at scheduled send time, not in real-time. There's a window where a touch could fire even though cancellation happened. Acceptable but document this caveat.

**Issue: Year-in-review math (Touch 04) shows generic defaults.**
- Helper flow to populate `bookings_count_membership_year` etc. isn't built or hasn't run yet.
- Build the helper flow OR fall back to hard-coded "average member" stats.

**Issue: Touch 05 always shows wrong tier.**
- Conditional split for Touch 05 isn't routing correctly. Verify `membership_tier` property values match split conditions exactly (case-sensitive — `Common User` not `Common user`).

**Issue: Renewal date is wrong on Touch 04 / Touch 05.**
- `renewal_date` property isn't being updated by Stripe sync. May need a helper flow to calculate it (membership_started_date + 365 days).

---

## Template note

All emails use **Flow / Personal**. The buildsheet recommends VIP/Status template for Touches 01 and 04 (premium dark treatment). If VIP/Status template doesn't exist yet, ship with Flow/Personal and upgrade later. Footer required:
- `RunRec` · `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8` · unsubscribe link · manage preferences link
