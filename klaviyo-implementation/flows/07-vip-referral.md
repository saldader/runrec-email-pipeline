# Flow 07 — VIP + Referral

**Build time estimate:** ~1.5 hours (4 touches initially + 1 quarterly recurring)
**Difficulty:** Medium-Hard (the location_manager merge tag is the franchise-architecture pivot — get it right)
**Prerequisites:**
- Pre-flight setup must be complete
- Sender domain `contact@therunrec.com` authenticated
- "Flow / Personal" master template AND "VIP / Status" premium dark template both exist
- Profile property `vip_since` exists (date) — written when the profile enters the VIP segment
- Profile properties `location_manager` (string) and `location_manager_email` (string) — these are franchise-critical. See deep dive at the bottom.
- "VIP / Power User" segment exists. Definition: profiles with 5+ Successfully Paid events in the last 90 days. Build this segment first if it doesn't exist.
- A unique referral link generator exists. The flow uses `runrec.co/r/{{first_name}}` — this short link should redirect to a tracked URL with the referrer's profile ID encoded. Talk to Eyad/Rob to confirm the setup.

**Klaviyo glossary** (terms used in this guide):
- **Status conferral** — the act of telling a customer they've earned a status (VIP, Regular, Insider) without asking for anything in return. Brunson "Attractive Character" tactic.
- **Earned ask** — making a request only after multiple value-only sends. The "ask" lands as natural, not transactional.
- **Quarterly recurring touch** — a touch that fires every 90 days as long as the profile remains in the VIP segment.

---

## What this flow does (plain English)

When a customer becomes a VIP (5+ bookings in 90 days), this flow sends them 5 touches over the first 30 days plus an ongoing quarterly drop:

1. **Status conferral** — "You're officially a regular. No tier, no fee. Just a few perks." This is the franchise pivot — sender + signature + contact email all use `{{location_manager}}` so any non-Waterloo VIP gets the local manager, not Sal.
2. **Insider drop** — "Early access to {{new_program}} for regulars only." Reinforces the VIP frame before any ask.
3. **The earned ask** — "Your friends play here too, right? No kickback program. Just a link." The referral request, framed as authentic.
4. **Frictionless share** — SMS with personalized referral link.
5. **VIP quarterly drop** — recurring every 90 days, members-only event/news/access.

The strategic point: **status conferral first, ask second.** VIPs don't respond to discounts — they respond to being seen. Two pure recognition touches (Touches 01 + 02) deposit value before the referral ask in Touch 03. By the time the ask lands, the relationship has earned it.

This flow also matters strategically because **referrals are the cheapest acquisition channel that exists.** A friend-to-friend SMS via Touch 04 converts at 5–8× the rate of cold outreach.

---

## Trigger — what fires this flow

- **Type:** Segment Joined
- **Segment:** VIP / Power User (5+ Successfully Paid events in last 90 days)
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name: `Flow 07 — VIP + Referral (v2 · 2026-05)`
  3. Trigger type: **Segment**
  4. Choose segment: **VIP / Power User**
  5. Trigger when: profile is added to segment

---

## Filters

- **Not currently in another active flow** — Klaviyo: "Was in any flow → Zero times in the last 30 days." Optional but reduces over-mailing risk during early launch.
- **Has email marketing consent = subscribed**

---

## Exit conditions

- **Drops out of VIP segment** — Klaviyo evaluates segment membership continuously. If the profile no longer has 5+ paid bookings in last 90 days (e.g., they slowed down), they fall out of the segment and exit this flow at the next conditional check. Quarterly recurring touch will not fire for ex-VIPs.
- **Unsubscribes** — automatic.

---

## Smart Sending: **OFF** — and why

VIPs are the most valuable segment in the database. Their messages should always reach. OFF on every touch.

## Quiet Hours: **ON for the SMS touch (Touch 04)**

8am–9pm recipient local.

---

## Flow map (visual)

```
TRIGGER: Segment Joined → VIP / Power User
   ↓
   Update Profile action: vip_since = today
   ↓
Touch 01 EMAIL — Trigger hit, immediate — Status conferral
   FROM: {{location_manager}} <{{location_manager_email}}>
   SIGNED: — {{location_manager}}
   ↓
Wait 10 days → 10am
   ↓
   Conditional: still in VIP segment?
   ├── YES → Touch 02 EMAIL — Insider drop / early access
   └── NO → EXIT
   ↓
Wait 10 days → 10am
   ↓
   Conditional: still in VIP segment?
   ├── YES → Touch 03 EMAIL — Referral ask
   └── NO → EXIT
   ↓
Wait 10 days → 3pm
   ↓
   Conditional: still in VIP segment?
   ├── YES → Touch 04 SMS — Frictionless share (personalized referral link)
   └── NO → EXIT
   ↓
Touch 05 EMAIL — Quarterly recurring (every 90 days while in segment)
   ↓
END (recurring)
```

---

## Touch-by-touch build instructions

### Touch 01 — Status conferral (Email) — THE FRANCHISE PIVOT

This touch is the most important in the flow because it establishes the multi-location pattern. **Get the merge tags right.**

- **Channel:** Email
- **Send delay:** Immediate on trigger
- **From name:** `{{ profile.location_manager|default:'The RunRec' }}`
- **From email:** `{{ profile.location_manager_email|default:'contact@therunrec.com' }}`
- **Reply-to:** same as From
- **Subject line:** `You're officially a regular.`
- **Preview text:** `No tier, no fee. Just a few perks for showing up.`
- **Template to use:** VIP / Status (premium dark — feels different from every other RunRec email)
- **Hero image:** Premium dark-mode treatment. Single line of accent-orange type on dark background. Or: hand-stamped "REGULAR" graphic.

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>You've booked 5+ sessions at RunRec in the last 90 days. That makes you a regular by our count.</p>
<p>There's no card. No tier. No fee.</p>
<p>But there are a few perks we extend to people who book this often:</p>
<p>→ <strong>Priority booking</strong> — first access to new programs and events before they go public<br>
→ <strong>Occasional surprise hours on the house</strong> (we'll text when we have an unexpected open slot)<br>
→ <strong>A direct line to me</strong> if anything ever goes sideways: {{ profile.location_manager_email|default:'contact@therunrec.com' }}</p>
<p>Welcome to the inner ring. We notice when you show up.</p>
<p style="font-style:italic;">— {{ profile.location_manager|default:'Sal' }}</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — A small fraction of the list books this often. You're in good company.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Update Profile** action right after the trigger. Set `vip_since` = `today`.
2. Drag an **Email** action.
3. Configure **From label** as: `{{ profile.location_manager|default:'The RunRec' }}` — this is a dynamic From label. Klaviyo supports merge tags in From label for many account tiers; if your tier doesn't, fall back to a static "The RunRec" From label and use the merge tag only in the body and signature.
4. Configure **From email** as: `{{ profile.location_manager_email|default:'contact@therunrec.com' }}` — same caveat. If dynamic From email isn't supported on your Klaviyo plan, fall back to `contact@therunrec.com` for the From email but keep the merge tag in the body.
5. Subject: `You're officially a regular.`
6. Preview: `No tier, no fee. Just a few perks for showing up.`
7. Edit content → paste body. Verify the merge tags `{{ profile.location_manager|default:'Sal' }}` and `{{ profile.location_manager_email|default:'contact@therunrec.com' }}` resolve correctly.
8. Smart Sending: OFF.
9. Save.

**The {{location_manager}} merge tag — what it does and why it matters:**
- For Waterloo (HQ): `location_manager = "Sal"`, `location_manager_email = "sal@therunrec.com"`. The email reads as "from Sal — Sal" — the local face that the customer would actually meet.
- For a future Burlington franchise: `location_manager = "Jamie"`, `location_manager_email = "jamie@therunrec.com"`. The same email reads as "from Jamie — Jamie" — the LOCAL franchisee, not Sal in Waterloo.
- This is the franchise-cloning pivot. Without these merge tags, every franchise would have to rebuild this flow from scratch with their own copy. With these merge tags, the flow is location-agnostic.

**Setting the merge tag values per profile:**
- The simplest approach: write `location_manager` and `location_manager_email` to every profile based on which physical location they primarily book. For Waterloo customers, default to Sal. For future locations, the local franchise's signup form or import process sets the value.
- Klaviyo segmentation can do this. Or a profile property update on Successfully Paid where the location is Waterloo sets `location_manager = "Sal"`.
- If both properties are blank for a profile (data gap), the fallback `|default:'Sal'` and `|default:'contact@therunrec.com'` keeps the email functional.

---

### Touch 02 — Insider drop / early access (Email)

- **Channel:** Email
- **Send delay:** Wait 10 days, send at 10am
- **From name:** `The RunRec` (per buildsheet doctrine, this touch is signed Sal)
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Early access — for regulars only.`
- **Preview text:** `{{ profile.new_program_name|default:"This quarter's program" }} launches public on {{ profile.launch_date|default:"soon" }}. You can book today.`
- **Template to use:** VIP / Status (dark, premium feel)
- **Hero image:** Hero image of the new program/event. Dark template treatment, accent details only.

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Quick one for the regulars.</p>
<p>We're launching <strong>{{ profile.new_program_name|default:"a new program" }}</strong> on {{ profile.launch_date|default:"the next week" }}.</p>
<p>Public sales open {{ profile.launch_date_plus_5|default:"5 days later" }}. <strong>You can book today.</strong></p>
<p><a href="{{ profile.program_details_url|default:'https://therunrec.com' }}">{{ profile.program_details_url|default:'therunrec.com' }}</a></p>
<p>First 20 spots only.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — We do these regular-first launches every couple of months. Worth keeping an eye on the inbox.</p>
```

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay**: "Wait 10 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: condition = "Properties about someone → Segment membership → is in segment → VIP / Power User."
   - YES → continue.
   - NO → EXIT.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

**When to skip this touch:** This touch only fires when there's a real new program/event to drop. If the operator doesn't have one this month, the merge tags will render as defaults and the email will be hollow. Two options:
- (a) Skip this touch entirely if `new_program_name` is blank. Add a conditional split: "Profile property `new_program_name` is set."
- (b) Send a generic "no specific drop this month" version. Not recommended — feels manufactured.

Operator decision. Recommended: option (a). Better to send fewer touches than fake exclusivity.

---

### Touch 03 — The referral ask (Email)

- **Channel:** Email
- **Send delay:** Wait 10 days, send at 10am
- **From name:** `The RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Your friends play here too, right?`
- **Preview text:** `No kickback program. Just a link.`
- **Template to use:** Flow / Personal (text-forward, no hero image)

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Honest question — your friends play sports too, right?</p>
<p>We've got a referral link for our regulars:</p>
<p><a href="https://runrec.co/r/{{ first_name|lower|default:'friend' }}">runrec.co/r/{{ first_name|lower|default:'friend' }}</a></p>
<p>When a friend uses your link, their first hour is free. <strong>You get nothing from us in return</strong> — no kickback, no points, no $5 off your next booking. We don't do those programs.</p>
<p>What you get is your friends playing here. That's the whole reward.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Our regulars who refer 2+ friends usually end up with a recurring crew that books a regular slot together. Cheaper, more fun, less hassle than coordinating one-offs.</p>
```

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02, drag a **Time Delay**: "Wait 10 days, send at 10:00 AM recipient local."
2. Drag a **Conditional Split**: VIP segment check. YES → continue. NO → EXIT.
3. Drag an **Email** action. Configure as above.
4. Smart Sending: OFF.

**The "you get nothing in return" hook is the brand-defining moment of this email.** Counter-positions against every gym referral program. Don't water this down. Operator must endorse.

The merge tag `{{ first_name|lower|default:'friend' }}` lowercases the first name for the URL slug. If this filter doesn't work in your Klaviyo plan, hardcode the URL pattern as `runrec.co/r/{{ first_name|default:'friend' }}` and accept mixed case.

---

### Touch 04 — Frictionless share (SMS)

- **Channel:** SMS
- **Send delay:** Wait 10 days, send at 3pm recipient local
- **Sender ID:** `RunRec`
- **Body content (paste exactly):**

```
Quick share: runrec.co/r/{{ first_name|lower|default:'friend' }} — your friend's first hour at RunRec is free.

-Sal Stop=opt-out
```

(Personalized link. One-tap forward. Texts share better than email forwards.)

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay**: "Wait 10 days, send at 3:00 PM recipient local."
2. Drag a **Conditional Split**: VIP segment check. YES → continue. NO → EXIT.
3. Drag a **Conditional Split**: SMS consent check.
4. Configure SMS: Sender RunRec, body as above, Quiet Hours ON, Smart Sending OFF.

---

### Touch 05 — VIP quarterly drop (Email — RECURRING)

- **Channel:** Email
- **Send delay:** Quarterly recurring, every 90 days as long as profile remains in VIP segment
- **From name:** `The RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `VIP drop — for regulars only.`
- **Preview text:** `Quarterly insider news. First access for 5 days.`
- **Template to use:** VIP / Status (dark, premium)
- **Hero image:** Premium dark-mode template. Hero image of the seasonal event/merch/new feature.

**Body content:**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Quarterly check-in for the regulars.</p>
<p>{{ profile.quarterly_drop_content|default:'[Insert seasonal content: new event, members-only night, merch drop, facility upgrade]' }}</p>
<p>First access for the next <strong>5 days</strong>. Public sales after that.</p>
<p><a href="{{ profile.drop_url|default:'https://therunrec.com' }}">{{ profile.drop_url|default:'therunrec.com' }}</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Last quarter's regular-only event ({{ profile.previous_event|default:'our members-only night' }}) sold out in 4 days. Heads up.</p>
```

**Klaviyo UI walkthrough — Touch 05 (the quarterly recurrence):**

Klaviyo flows don't natively support "recurring every 90 days within the same flow." The standard pattern is:
- (a) **Build a separate Klaviyo flow** triggered on a date-property like `vip_quarterly_drop_due` that you set to "today + 90 days" each time the email fires.
- (b) Or use Klaviyo Campaigns scheduled to send to the VIP segment every quarter manually.
- (c) Or build the recurrence as a self-looping wait: after Touch 05 fires, add a "Wait 90 days" → loop back to a conditional check (still in VIP segment?) → re-send Touch 05. Klaviyo allows this, though the flow visualization gets ugly.

Recommended: option (b) for v1. Simplest. Operator manually launches a quarterly campaign to the VIP segment using the Touch 05 body content as a campaign template.

If going with option (a) or (c), make sure the recurrence stops if the profile drops out of the VIP segment.

---

## Test plan

1. Create a test profile. Set `location_manager = "Sal"`, `location_manager_email = "sal@therunrec.com"`, and add 5 fake `Successfully Paid` events in the last 90 days (or manually add to the VIP / Power User segment).
2. Touch 01 should fire within hours. Verify:
   - From label resolves to `Sal` (or whatever the manager is)
   - From email resolves to `sal@therunrec.com`
   - Body signature resolves to `— Sal`
   - The "direct line" email line in the body resolves to `sal@therunrec.com`
3. Test the multi-location pivot: change the profile to `location_manager = "Jamie"`, `location_manager_email = "jamie@therunrec.com"`, then re-trigger. Verify Touch 01 reads as "from Jamie — Jamie" with `jamie@therunrec.com` as the contact.
4. Use "Send next message immediately" to fast-forward through Touches 02–04.
5. Verify the personalized referral link in Touch 03 and Touch 04 renders correctly.
6. Verify the conditional VIP segment check exits the flow if you remove the profile from the segment.

**Edge cases:**
- Profile with blank `location_manager` → fallback to "The RunRec" / "contact@therunrec.com". Verify fallbacks work.
- Profile with no SMS consent → Touch 04 skipped, but other touches send.
- Profile drops out of VIP segment between Touch 02 and Touch 03 → flow exits, no Touch 03 sent. Verify.

---

## Activation checklist

- [ ] All 4 initial touches built (Touches 01–04)
- [ ] Touch 05 recurrence mechanism decided (separate flow, manual campaign, or self-loop) and built
- [ ] VIP / Power User segment exists and is correctly defined (5+ Successfully Paid in 90 days)
- [ ] Profile properties `location_manager` and `location_manager_email` populated for Waterloo customers (default values: "Sal" / "sal@therunrec.com")
- [ ] Touch 01 dynamic From label and From email tested with both Waterloo and a fake non-Waterloo profile
- [ ] Conditional splits on Touches 02, 03, 04 check VIP segment membership
- [ ] SMS consent check on Touch 04
- [ ] Referral link `runrec.co/r/{{ first_name }}` setup is working (URL redirects, tracks referrer, gives friend first hour free)
- [ ] Quiet Hours ON for Touch 04 SMS
- [ ] Smart Sending OFF on every touch
- [ ] "VIP / Status" template exists and is correctly styled (premium dark)
- [ ] Footer compliance on every email
- [ ] `vip_since` profile property is set at flow entry via Update Profile action
- [ ] Test profile has been through full flow with both Waterloo and franchise variants verified

---

## Post-launch monitoring (first 90 days)

- **Touch 01 open rate:** Target 70%+ (VIPs are the most engaged segment; "you're a regular" emails always have very high opens).
- **Touch 01 reply rate:** Target 5%+. Replies to status conferral are usually thank-yous or feedback — surface to Sal/Izzy for personal response.
- **Touch 02 click-through rate:** Depends on the program. 15%+ is healthy.
- **Touch 03 open rate:** Target 60%+. The referral ask has a curiosity-driven subject line.
- **Touch 04 SMS click-through rate:** Target 25%+. SMS share rates are very high among engaged customers.
- **Referrals generated (number of new signups attributed to a VIP's link):** This is the headline number. Track via the URL parameter on `runrec.co/r/{{first_name}}`. Target: 0.5–1.5 referrals per VIP per quarter.
- **Touch 05 quarterly drop click-through:** Target 30%+ since these are the most engaged customers receiving the most curated content.

**Warning signs:**
- VIP STOP rate above 0.1% — VIPs unsubscribing means the perks aren't real or the cadence is too aggressive. Investigate.
- Referral link clicked but no signup — landing page friction. Walk through the URL → signup flow yourself to find the problem.
- Touch 02 sent but program details URL is broken — operator should verify the URL is set before each launch.

---

## Common issues for THIS flow

1. **The {{location_manager}} merge tags are the franchise pivot — without them, this flow doesn't clone.** Get them right at Touch 01. Fallbacks to "Sal" / "sal@therunrec.com" are fine for Waterloo HQ but the template is now multi-location-ready.

2. **Touch 02 fires regardless of whether there's actually a new program.** Without conditional logic to skip empty drops, this email becomes hollow. Add the "if `new_program_name` is set" conditional or skip Touch 02 entirely some quarters.

3. **The "you get nothing in return" hook in Touch 03 is the brand differentiator.** Don't let copy reviewers dilute it to "you get the joy of sharing" or "you get good karma." The directness IS the appeal. Most referral programs offer kickbacks; not offering one is the counter-positioning.

4. **The personalized referral link `runrec.co/r/{{first_name}}` requires URL infrastructure.** If two VIPs are both named "Sarah," they'll have the same URL. Better to use `runrec.co/r/{{external_id}}` or `runrec.co/r/{{profile_id}}`. Operator decision: simpler URL with collision risk OR cleaner URL that's harder to remember/share. Recommended: use first_name + last initial: `runrec.co/r/{{first_name|lower}}-{{last_name_initial|lower}}` — readable AND collision-resistant.

5. **The VIP segment definition matters.** "5+ Successfully Paid in 90 days" is a high bar. Currently the Klaviyo account has zero segments built besides "Everyone" and "New Subscribers" — so the VIP segment must be built first. Without it, this flow has no audience.

6. **Quarterly recurring touch is hard to build natively.** Klaviyo flows don't loop cleanly. The cleanest implementation is a separate quarterly Campaign sent to the VIP segment manually. Loops in flow canvases work but are visually confusing and easy to misconfigure.

7. **VIP fall-out is silent.** A profile that drops from 6 bookings/90-day to 3 bookings/90-day silently exits the VIP segment, which exits this flow at the next conditional. They DON'T get a "you've lost VIP status" email. That's intentional — sad-news emails don't drive engagement. But the operator should be aware.

8. **The flow can over-mail engaged customers.** A VIP customer might also be in Welcome (if recent signup), Booking Confirm (every booking), Post-Session (every booking), AND Win-Back (if they slow down). Coordinate cadence: a VIP booking 6 sessions in 90 days could receive 50+ messages from RunRec in the same period. Watch for unsubscribes.

9. **Sender doctrine inconsistency in the deployed dashboard.** Touch 01 currently shows "From: The RunRec" (not `{{location_manager}}`) and signs "— Sal." This is the canonical example of the franchise-pivot work that wasn't fully completed. The buildsheet's target state is `{{location_manager}}` end-to-end. Pick one before launch and apply consistently. Recommended: apply the merge tags as documented above, with Sal/sal@ as Waterloo defaults.
