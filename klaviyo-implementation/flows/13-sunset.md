# Flow 13 — Sunset / Hygiene

**Build time estimate:** 2 hours (only 2 touches, but the segment definition + auto-suppress logic are critical to get right)
**Difficulty:** Medium (mechanically simple, but the consequences are PERMANENT — profiles get unsubscribed forever if they don't engage)
**Prerequisites:** Pre-flight setup complete, plus the items below.

### What needs to exist BEFORE you build this

- The `Unengaged 90+ Days` segment (we will create this in step 1).
- A "Stay subscribed" landing page or one-click action URL (`therunrec.com/stay-subscribed` — clicking confirms continued opt-in).
- A "monthly only" subscription preference URL (`therunrec.com/email-preferences/monthly`) — gives sunset profiles a downgrade option instead of full unsubscribe.
- Profile property `unengaged_sunset_complete` (boolean) — used by the auto-suppress action at Day 14.

---

## What this flow does (plain English) — READ THIS CAREFULLY

This flow sends 2 emails to profiles who haven't opened or clicked an email in 90+ days. After 14 days, anyone who STILL hasn't engaged gets **automatically unsubscribed (suppressed) from the email list, permanently.**

The flow's job: protect deliverability. Dormant subscribers tell Gmail/Apple "this sender's emails aren't worth opening" — which hurts inbox placement for ALL future emails to ALL recipients. Counter-intuitively: aggressive list hygiene IMPROVES inbox placement for engaged subscribers.

**Why this is permanent:** Once Klaviyo suppresses a profile, the profile cannot be re-added to a list automatically. They can re-subscribe by clicking a signup form on the website, but they will not receive any marketing email until they do. This is intentional. Suppressed profiles can be reviewed and manually unsuppressed by an admin, but the default behavior is permanent.

**Operator: confirm you understand this before activating.** This is the only flow in the system that REMOVES customers from your reachable audience. The trade-off: the customers it removes weren't reachable anyway (90+ days no opens). Better to clean them out than to keep paying Klaviyo for them and dragging down deliverability.

---

## Trigger — what fires this flow

- **Type:** Segment-triggered (when a profile enters the `Unengaged 90+ Days` segment)
- **Why segment-triggered:** Trigger needs multiple conditions — engagement, list tenure, send history.

### Step 1 — Create the trigger segment (do this BEFORE building the flow)

In Klaviyo: **Audience → Segments → Create Segment.**

- **Name:** `Unengaged 90+ Days`
- **Definition (read every condition carefully):**

  Condition 1 — **Engagement check (the core):**
  - "What someone has done (or not done)"
  - Pick metric: `Opened Email` OR `Clicked Email` (combine both with OR)
  - Set: **has done zero (0) times** in the last **90 days**
  
  Actually, easier to express this as:
  - "What someone has done (or not done)"
  - Pick metric: `Opened Email`
  - Set: has done **zero (0) times** in the last **90 days**
  - AND ANOTHER condition:
  - Pick metric: `Clicked Email`
  - Set: has done **zero (0) times** in the last **90 days**

  Both must be true (zero opens AND zero clicks in 90 days).

  Condition 2 — **List tenure check (don't sunset brand-new subscribers):**
  - "Properties about someone"
  - Pick: `Subscribed to Email Marketing` date (or use a `Subscribed to List` event check)
  - Set: more than **180 days ago** (so we only sunset people who've been on list 6+ months — gives them a fair chance)

  Condition 3 — **Send history check (don't sunset profiles we never emailed):**
  - "What someone has done (or not done)"
  - Pick metric: `Received Email`
  - Set: has done **at least 5 times** in the last **72 weeks**

  Condition 4 — **Currently subscribed:**
  - "Properties about someone"
  - Pick: `Email Marketing Consent` (or "Is in list" → Newsletter list `SDFmeU`)
  - Set: equals **subscribed**

  Condition 5 — **Not currently in any other active flow** (prevent collision):
  - This is hard to express directly. Best approximation: NOT in segment `Members Active` (sunset shouldn't fire on paying members) AND NOT in segment `Conversion Eligible` (don't sunset people about to convert).

- Click Create Segment.

**Klaviyo will start building the segment.** For accounts with 3,000 profiles, this takes 5-15 minutes. For larger accounts, longer. Watch the segment count populate. The expected count for a healthy newsletter list is 30-50% of total — for RunRec's ~3,000 marketable profiles, expect 900-1,500 in this segment at launch.

**WARNING:** This segment will be your largest audience cohort. Confirm with operator before activating Flow 13 — they're about to email 1,000+ people and (potentially) suppress most of them.

---

## Filters

In flow trigger configuration:

- **Filter 1:** Profile is in segment `Unengaged 90+ Days` (yes — duplicate of trigger as safety check)
- **Filter 2:** Profile is NOT in any other active flow. (Klaviyo doesn't natively support this. Approximation: NOT in `Members Active` AND NOT in `Conversion Eligible`.)
- **Filter 3:** Email marketing consent = `subscribed`.
- **Filter 4:** `unengaged_sunset_complete` does NOT equal `true` (prevents re-trigger if they were sunset before and somehow re-entered the segment).

---

## Exit conditions

- **Exit if:** Opens or clicks ANY email (this auto-removes them from the segment, exits flow cleanly — they re-engaged, save them).
- **Exit if:** Email marketing consent = `unsubscribed` (clean exit).
- **Exit at end of flow:** After Touch 02 + 7 days, the auto-suppress action fires (see below).

---

## Smart Sending: ON

**Why ON:** These are unengaged profiles. Don't compound the problem by mailing them when they recently received another email. Smart Sending will skip if they got any other Klaviyo email in last 16 hours.

## Quiet Hours: 9 AM – 7 PM recipient timezone

Standard hours. No special rules.

---

## Flow map (visual)

```
Trigger: joins Unengaged 90+ Days segment
  ↓
Filters
  ↓
Touch 01 (Day 0) → Email: Are you still in?
  ↓ Wait 7 days
Touch 02 (Day 7) → Email: Last email
  ↓ Wait 7 days
AUTO-SUPPRESS ACTION: tag unengaged_sunset_complete = true → unsubscribe profile
  ↓
EXIT
```

---

## Touch-by-touch build instructions

### Touch 01 — Are you still in?

- **Channel:** Email
- **Send delay:** Day 0 — fires immediately on segment entry
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Reply-to:** contact@therunrec.com
- **Subject line:** `Are you still in?`
- **Preview text:** `Click YES to stay or unsubscribe — no hard feelings.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>You haven't opened or clicked an email from us in 90+ days.</p>

<p>We don't want to clutter your inbox. Two options:</p>

<p><strong>1.</strong> Click YES to keep getting RunRec updates: <a href="https://therunrec.com/stay-subscribed?profile={{ person.id }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Stay subscribed</a><br>
<strong>2.</strong> Unsubscribe (no hard feelings): <a href="{{ unsubscribe }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Unsubscribe</a></p>

<p>If we don't hear from you, we'll automatically remove you from the list in 14 days.</p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you want to stay subscribed but get fewer emails, click here to switch to monthly-only: <a href="https://therunrec.com/email-preferences/monthly?profile={{ person.id }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Switch to monthly</a></p>
```

#### How the "Stay subscribed" click works

The link goes to `therunrec.com/stay-subscribed?profile={{ person.id }}`. The landing page must:
1. Capture the profile ID from URL.
2. Call Klaviyo's API to fire a `Stayed Subscribed` event for that profile.
3. Show a "You're staying subscribed!" confirmation page.

Klaviyo's exit condition ("Opens or clicks any email") will already catch this click and exit the flow. The custom event is optional but useful for tracking how many people consciously chose to stay.

#### How "Switch to monthly" works

Same architecture — landing page captures profile ID, sets a property `email_frequency = monthly`, fires a `Downgraded to Monthly` event. Then sets a flow to remove them from regular newsletter sends and only include them in monthly digest sends.

For launch, the monthly preference page can just say "Got it — you'll receive fewer emails" and update the property. Build out the actual frequency control later.

#### Klaviyo UI walkthrough for Touch 01

1. From the empty flow canvas (after trigger), click **"+"** → **Email**.
2. Name: `Flow 13 — T01 — Are You Still In`.
3. **Set delay BEFORE this email to `0 seconds`** (fires immediately on segment join).
4. From name, email, reply-to as above.
5. Subject + preview text.
6. Edit content → Flow / Personal template → paste body HTML.
7. Smart Sending: ON.
8. Quiet hours: 9 AM – 7 PM recipient timezone.
9. Save.

---

### Touch 02 — Last email

- **Channel:** Email
- **Send delay:** Wait `7 days` after Touch 01 → Day 7 total
- **From name:** RunRec
- **From email:** contact@therunrec.com
- **Subject line:** `Last email unless you say otherwise.`
- **Preview text:** `Day 14: auto-suppress. Just data hygiene.`
- **Template:** Flow / Personal

#### Body content

```html
<p>Hey {{ first_name|default:'there' }},</p>

<p>This is the last email you'll get from us unless you click below to stay on the list.</p>

<p><a href="https://therunrec.com/stay-subscribed?profile={{ person.id }}" style="color:#e85a3e;text-decoration:underline;font-weight:600;">Stay subscribed</a></p>

<p>After 7 more days, we'll auto-suppress your profile to protect deliverability for everyone else on the list.</p>

<p>No hard feelings. <strong>Just data hygiene.</strong></p>

<p style="font-style:italic;margin-top:20px;">— RunRec Team</p>

<hr style="border:none;border-top:1px dashed rgba(26,26,26,0.18);margin:20px 0;" />

<p style="color:#2a2a2a;font-size:13px;"><strong style="color:#e85a3e;">PS</strong>, If you ever want to come back, you can re-subscribe anytime at therunrec.com/subscribe.</p>
```

---

### After Touch 02 — THE AUTO-SUPPRESS ACTION (7 more days)

This is the critical step. After Touch 02 + 7 days, profiles who STILL haven't opened/clicked get suppressed.

#### Klaviyo UI walkthrough for the suppress action

1. After Touch 02, drag in **Time Delay → 7 days**.
2. Click **"+"** → look for action type **"Update Profile Property"** (or similar).
3. Name: `Flow 13 — Sunset Tag`.
4. Action: set property `unengaged_sunset_complete = true`.
5. Drag in **another action**: this requires either:
   - **Klaviyo native "Unsubscribe Profile" action** (if available in your plan — newer feature).
   - OR **Webhook action** that calls Klaviyo's own API to suppress the profile (POST to `/api/profile-suppression-bulk-create-jobs/`).
6. Configure the unsubscribe action to suppress from email marketing.
7. Save and end flow.

**If your Klaviyo plan doesn't support in-flow profile suppression:**

Build a daily helper job (outside Klaviyo, e.g., a cron-triggered Python script using the Klaviyo MCP `klaviyo_profiles_suppress` tool). The script runs daily, finds all profiles with `unengaged_sunset_complete = true`, and suppresses them via API.

In flow: just set the property tag. Outside flow: daily script handles the actual suppression.

For launch, set the property + ALSO configure a daily review where operator manually suppresses tagged profiles. Build the automated job in v2.

---

## Test plan

Before activating:

1. **Verify segment populates correctly.** Check the count of `Unengaged 90+ Days`. Spot-check 3-5 profiles in it — confirm none have opened/clicked recently.
2. **Create a test profile** matching segment criteria: been on list 180+ days, 5+ emails received, zero opens/clicks in 90 days. (Easiest: create profile, force-add to Newsletter list with backdated subscription, send 5 test emails to a real address that you let bounce/ignore for 91 days. Or: manually add to segment via Klaviyo segment override.)
3. **Verify Touch 01 fires within 1 hour of segment join.**
4. Open Touch 01 — verify "Stay subscribed" link works AND triggers the exit condition (open click should remove profile from segment, exit flow).
5. **Test the "Stay subscribed" exit:** click the link from a test profile. Verify within 24 hours the profile exits the flow.
6. **Test the "Last email" Touch 02** — manually advance test profile through Touch 01 with no engagement. Verify Touch 02 arrives 7 days later.
7. **Test the auto-suppress action.** Advance test profile through both touches with no engagement. Verify after Day 14:
   - `unengaged_sunset_complete` = true on profile
   - Profile is in suppression list
   - Profile cannot receive future emails
8. **Test the engagement reset:** create another test profile that opens Touch 01 within 5 days. Verify the profile EXITS the flow and is NOT suppressed.

---

## Activation checklist — STOP AND READ EVERYTHING

This is a destructive flow. Mistakes are permanent. Get every checkbox right.

- [ ] Operator has explicitly approved auto-suppression behavior. (This is mandatory. Do not proceed without explicit "yes, suppress 90-day-unengaged profiles" from operator.)
- [ ] Segment `Unengaged 90+ Days` built and populated
- [ ] Segment count is reviewed — operator knows the number of profiles that will receive Touch 01 in the first 24 hours
- [ ] "Stay subscribed" landing page is live at `therunrec.com/stay-subscribed`
- [ ] "Switch to monthly" landing page is live (or stub page that just acknowledges the request)
- [ ] Touch 01 + Touch 02 built and tested
- [ ] Auto-suppress mechanism configured (in-flow OR daily helper job)
- [ ] Suppression review SOP in place (operator reviews weekly suppress list before they go permanent)
- [ ] Smart Sending ON for both touches
- [ ] Sender Profile `RunRec` exists (for the From: line)
- [ ] **PILOT TEST:** activate flow on a small sub-segment first (e.g., manually narrow segment to 50 profiles for week 1, expand to full segment after seeing results)
- [ ] Operator approval to activate at full scale
- [ ] Backup of suppressed-profile list exported before activation (so we can recover if there's a bug)

---

## Post-launch monitoring (DAILY for first 14 days)

This flow has the highest "could go wrong" potential of any flow. Monitor obsessively.

- **Touch 01 entry volume.** Match against segment count — should be roughly equal in the first 24 hours.
- **Touch 01 open rate.** Target 8-15% (these are unengaged — opens will be low). If you get 30%+, something's wrong with the segment definition (you're sunsetting engaged people).
- **"Stay subscribed" click rate.** Target 5-10% — these are people you saved. The PS "switch to monthly" option may catch additional saves.
- **Unsubscribe rate (active).** Target 5-15% click unsubscribe directly. This is HEALTHY — you're surfacing low-value subscribers and they're self-cleaning.
- **Touch 02 entry volume.** Should be ~85-95% of Touch 01 volume (the rest exited via opens/clicks/unsubscribes).
- **Auto-suppression count at Day 14.** Compare against pre-flow segment count. Healthy: 60-80% of original segment gets suppressed.
- **Total list size BEFORE vs AFTER first wave.** Expect 20-40% list reduction. This is the goal. Operator must understand this is a feature, not a bug.
- **Engaged-subscriber open rate AFTER first wave.** Should INCREASE over the following 30 days. If your engaged-segment open rate jumps from 35% to 45%+, the sunset worked.
- **Errors in suppress action.** Check Klaviyo flow execution log for any failures on the auto-suppress step. Failed suppressions create the "should be suppressed but still receiving emails" bug.

---

## Common issues for THIS flow

**Issue: Segment count is way too high (e.g., 80% of total list).**
- The list has been poorly maintained for years OR the segment definition is wrong.
- Verify the "received 5+ emails in 72 weeks" condition — without it, you'd be sunsetting profiles you never actually emailed.
- Consider phasing the rollout: shrink to 90+ AND received 10+ emails AND on-list 365+ days for the first wave; expand criteria gradually.

**Issue: Engaged customers receive Touch 01.**
- Segment definition is broken. Most likely: the open/click condition is OR instead of AND, or the time window is wrong.
- IMMEDIATE ACTION: pause the flow. Audit the segment. Send an apology email to anyone who received Touch 01 in error.

**Issue: Auto-suppress isn't firing.**
- In-flow suppression action isn't supported in your Klaviyo plan. Build the daily helper script.
- Or: profile property `unengaged_sunset_complete = true` is being set but the script that reads it isn't running.

**Issue: Suppressed profile somehow receives an email later.**
- Suppression isn't being applied account-wide. Check Klaviyo Suppression list — is the profile actually in it?
- Or: profile has manual override on a list. Check list-level suppression vs account-level.

**Issue: Customer says "I want to come back."**
- Operator manually unsuppresses via Klaviyo Profiles → Suppressions → Remove.
- Send them to `therunrec.com/subscribe` to re-opt-in cleanly with a fresh consent timestamp.

**Issue: Operator panics about list size shrinking.**
- This is the goal. Rebuild trust by showing engaged-segment open rate going UP.
- Counter-message: "We removed people who weren't reading our emails anyway. The list is now smaller but reaches more inboxes."

---

## Final note on timing

The buildsheet recommends activating Flow 13 LAST among all flows — only after 90+ days of clean data from the other 12 flows. Reasoning: you need engagement data to identify "unengaged 90+ days" — and that data comes from sending emails the profiles either open or don't.

If you activate Flow 13 too early (before the other flows are running), the segment will be inaccurate (some "unengaged" profiles haven't been engaged because we haven't been emailing them recently).

**Recommended order:**
1. Build all 12 other flows first
2. Run them for 90 days (collect engagement data)
3. Then build Flow 13
4. Then activate Flow 13 in pilot mode (small segment)
5. Then activate at full scale

This is the slowest flow to ship but the one most worth getting right.

---

## Template note

Both emails use **Flow / Personal**. Footer required and especially important here:
- `RunRec` · `283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8` · unsubscribe link · manage preferences link

The unsubscribe link in BOTH the body AND footer must work. Test before activating. Sunset emails are the highest-scrutiny emails Gmail reads — broken unsubscribe links will tank deliverability further.
