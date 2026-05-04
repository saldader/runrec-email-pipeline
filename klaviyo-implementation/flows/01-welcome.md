# Flow 01 — Welcome Series

**Build time estimate:** ~2 hours
**Difficulty:** Medium (it's the longest flow, but most touches are the same pattern)
**Prerequisites:**
- Pre-flight setup must be complete (see `~/vault/runrec/marketing/klaviyo-implementation/00-current-klaviyo-state.md`)
- Sender domain `contact@therunrec.com` must be authenticated (DKIM/SPF/DMARC) — confirm in Klaviyo → Settings → Domains
- "Newsletter" list (ID: `SDFmeU`) must exist (it already does — it's the de-facto main list)
- "Flow / Personal" master template should exist (if not, build it before starting — see template specs at the bottom of this doc)
- Profile property `player_type` must exist (string) — this is what Touch 05 sets via the click links. Create it under Settings → Profile properties before launch
- The existing live "2025 - Welcome Flow (Visual Email)" flow `RZsSVX` already exists. **Decision needed before you start:** are you (a) extending that flow to match this 7-touch spec, or (b) building this as a brand new flow and pausing/archiving `RZsSVX`? Most operators choose option (b) — it's cleaner. This guide assumes option (b).

**Klaviyo glossary** (terms used in this guide):
- **Flow** — Klaviyo's word for an automated email sequence triggered by an event.
- **Trigger** — the thing that starts the flow (e.g., joining a list, placing an order).
- **Filter** — a rule that decides if a profile is allowed to enter the flow.
- **Touch** — one message in the flow. Can be email or SMS.
- **Time delay** — a wait step between touches. Klaviyo calls it a "Wait" action.
- **Smart Sending** — a Klaviyo setting that suppresses a send if the profile got an email/SMS within the last 16 hours. Useful for campaigns; **dangerous for flows** because it skips the entire touch (the profile doesn't catch up later).
- **Quiet Hours** — Klaviyo setting that holds an SMS until morning if it's scheduled outside the recipient's local 8am–9pm window.
- **Merge tag** — a placeholder in copy that fills in with profile data, e.g., `{{ first_name }}`. Klaviyo's syntax uses double curly braces.

---

## What this flow does (plain English)

When someone joins the Newsletter list (via the website footer, a popup, or a manual import), this flow welcomes them over 14 days with 6 emails and 1 SMS. The job is not to sell on Day 1 — it's to plant the brand identity (24/7 private courts, no staff, premium), tell the founder story, hand over a few useful insider tips, and then ask for the first booking on Day 14 once the relationship has earned the ask. By Day 14 the profile has either booked (and exits to Booking Confirm + Post-Session) or is unconverted (and rolls into Lead Nurture eventually).

This is the foundation of acquisition. It's the longest, most carefully-paced flow in the system. Every other flow assumes this one runs cleanly first.

---

## Trigger — what fires this flow

- **Type:** Added to List
- **Specific config in Klaviyo:**
  1. Open Flows → Create Flow → Create from Scratch
  2. Name the flow: `Flow 01 — Welcome Series (v2 · 2026-05)`
  3. Select trigger type: **List**
  4. Choose list: **Newsletter** (ID `SDFmeU`)
  5. Set "When someone is added" (this is the default and what you want)

---

## Filters

Click "Add filter" on the trigger card and add each of these:

- **Has email marketing consent = subscribed** (Klaviyo will call this "Properties about someone → Email Consent → equals → Subscribed")
- **Has not been in this flow in the last 90 days** (Klaviyo: "Was in this flow → Zero times in the last 90 days")
- **Optional but recommended:** Filter out profiles already in the Skedda Email List (`WyA26L`). They're already paying customers — don't run them through the indoctrination sequence. Klaviyo: "Properties about someone → List membership → is not in list → Skedda Email List."

---

## Exit conditions

Click the trigger → "Flow exits" tab → add:

- **Books first session** — Klaviyo: "Has done event → Successfully Paid → at least once since starting this flow." This moves them out of Welcome and into Flow 02 (Booking Confirm) and Flow 03 (Post-Session).
- **Unsubscribes from email** — Klaviyo handles this automatically when "Email Consent" leaves "Subscribed."
- **Marks as spam** — Klaviyo handles automatically.

---

## Smart Sending: **OFF** — and why

Welcome flows MUST complete in order. If you turn Smart Sending ON and the profile happens to be in another flow (very common for new signups), Klaviyo will skip the welcome touches entirely and the relationship building dies on the vine. Always OFF for welcome flows.

How to turn off: After you save the flow, click the gear/settings icon at the top → uncheck "Smart Sending" for each individual email and SMS touch. You may have to do this per-touch.

## Quiet Hours: **ON for SMS only**

For the email touches: leave default (no quiet hours).
For the SMS touches (Touches 04 and 07): turn Quiet Hours ON, set window to 8am–9pm in the recipient's local timezone. Klaviyo will hold the send until the next valid window if scheduled outside it.

---

## Flow map (visual)

```
TRIGGER: Added to Newsletter list
   ↓
   (Filter check: consent + not in flow recently + not already a Skedda customer)
   ↓
Touch 01 EMAIL — Day 0 immediate — "What you just signed up for."
   ↓
Wait 1 day → 10am ET
   ↓
Touch 02 EMAIL — Day 1 10am ET — "The real reason we built this place."
   ↓
Wait 2 days → 10am ET
   ↓
Touch 03 EMAIL — Day 3 10am ET — "The 11pm court hack nobody tells you."
   ↓
Wait 2 days → 1pm ET
   ↓
Touch 04 SMS — Day 5 1pm ET — warmth check
   ↓
Wait 2 days → 10am ET
   ↓
Touch 05 EMAIL — Day 7 10am ET — "Which type of player are you?" (4 click options)
   ↓
   (Click on link sets profile property `player_type` = solo / crew / party / league)
   ↓
Wait 3 days → 10am ET
   ↓
Touch 06 EMAIL — Day 10 10am ET — "What happens here at 2am."
   ↓
Wait 4 days → 11am ET
   ↓
Touch 07 SMS — Day 14 11am ET — the close
   ↓
END (profile exits cleanly; if not converted, rolls into Lead Nurture per Flow 08 spec)
```

There are NO conditional branches in this flow. Every profile gets every touch in order until they either convert (exit via Successfully Paid) or finish the flow.

---

## Touch-by-touch build instructions

### Touch 01 — Welcome & indoctrination

- **Channel:** Email
- **Send delay:** Immediate (no wait — fires on trigger)
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `What you just signed up for.`
- **Preview text:** `24/7 private courts. No staff. No crowds. Just play.`
- **Template to use:** Flow / Personal
- **Hero image:** Wide-angle hero shot of empty Court A at 11pm — overhead lights on, basketball at center court, no people. Slightly moody, warm tones. Logo bottom right. (16:9 ratio, 1200×675px)

**Body content (paste this HTML directly into the Klaviyo email editor's HTML view):**

```html
<p>Hey {{ first_name|default:'there' }},</p>
<p>Welcome to RunRec.</p>
<p>You just joined a club where the courts are open at 2am if that's when your crew can play.</p>
<p>No front desk. No waiting. No "sorry that hour is taken" — because the hour is yours alone when you book it.</p>
<p><strong>Three full courts. Four sports. 24 hours a day, 365 days a year.</strong></p>
<p>That's the whole pitch. The rest is logistics.</p>
<p>When you're ready to book, the courts are at <a href="https://therunrec.com/book">therunrec.com/book</a>.</p>
<p style="font-style:italic;">— Sal &amp; Izzy – RunRec Co-Owners</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Drag this email to your Primary tab so we don't end up in Promotions. The good stuff is coming.</p>
```

**Klaviyo UI walkthrough — Touch 01:**
1. From the flow canvas, drag an **Email** action to the canvas right after the trigger.
2. Click the email block → "Configure content."
3. Fill the From label, From email, Subject, Preview text exactly as above.
4. Click "Edit content" → if the template uses drag-drop, switch the body block to "Custom HTML" mode and paste the body content above. Make sure to remove any default placeholder text in the template body.
5. Add the unsubscribe footer block at the bottom (the template should have one — verify it includes "RunRec · 283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8 · unsubscribe · manage preferences").
6. Save → Set status to "Live."
7. Smart Sending: OFF for this touch (gear icon → uncheck).

**Conditional logic:** None.

---

### Touch 02 — Origin story

- **Channel:** Email
- **Send delay:** Wait 1 day, then send at 10am ET (use the "Time delay" action set to "Until specific time of day" → 10:00 AM Eastern Time)
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `The real reason we built this place.`
- **Preview text:** `It started with a problem nobody else was solving.`
- **Template to use:** Flow / Personal
- **Hero image:** Casual side-by-side photo of Sal & Izzy on the court — not posed, mid-conversation, evening lighting. Real founders shot, not corporate headshot. Or alternative: hand-drawn graphic showing "9pm CLOSED" on every other gym vs RunRec "OPEN 24/7."

**Body content:**

```html
<p>Two years ago we couldn't find a court.</p>
<p>Every gym in Waterloo closed at 9pm. Every YMCA was packed at 6pm. Every "rec center" had four teenagers in line ahead of us and no clue when a hoop would open.</p>
<p>We just wanted to play. Privately. On our own time.</p>
<p>Nobody had built that.</p>
<p>So we did.</p>
<p>RunRec is a 24/7 unstaffed multi-sport facility because that's the place we wished existed when we were trying to find a court at 10pm on a Tuesday.</p>
<p>If you've ever been turned away from a public gym, you'll get it.</p>
<p style="font-style:italic;">— Sal &amp; Izzy – RunRec Co-Owners</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Most of our regulars book between 9pm-12am. Turns out a lot of people had the same problem we did.</p>
```

**Klaviyo UI walkthrough — Touch 02:**
1. After Touch 01, drag a **Time Delay** action onto the canvas. Configure: "Wait 1 day, then send at 10:00 AM" → timezone "Recipient's local timezone, fall back to America/Toronto."
2. Drag an **Email** action after the delay. Configure as above.
3. Smart Sending: OFF.

---

### Touch 03 — Pure value content

- **Channel:** Email
- **Send delay:** Wait 2 days from Touch 02, send at 10am ET
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `The 11pm court hack nobody tells you.`
- **Preview text:** `3 things our regulars do that first-timers miss.`
- **Template to use:** Flow / Personal (text-forward — no hero image)
- **Hero image:** None. This email reads like a friend's tip text. Optional: 3 small inline icons (32px each) — basketball, clock, speaker — beside each of the 3 numbered tips. If you don't have those icons, skip the icons entirely.

**Body content:**

```html
<p>After thousands of court bookings, we've noticed the regulars do three things first-timers don't:</p>
<p><strong>1. For half-court basketball, book Court B or C.</strong> They're $45/hr instead of $89. Same hardwood, same hoops. (Other sports use Court A only.)</p>
<p><strong>2. Off-peak hours (11pm-7am) are wide open.</strong> The 24/7 thing isn't a gimmick — it's a hack. Completely private feel.</p>
<p><strong>3. Bring a Bluetooth speaker.</strong> The acoustics in here are unreal. Most people don't realize it until the third visit.</p>
<p>That's the playbook. Use it whenever you book.</p>
<p>When you're ready: <a href="https://therunrec.com/book">therunrec.com/book</a></p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — One more: the door code is tied to your booking. You'll get a text 2 hours before your session with the code. Don't show up early — there's no front desk to let you in.</p>
```

**Klaviyo UI walkthrough — Touch 03:**
1. After Touch 02, drag a **Time Delay** action: "Wait 2 days, send at 10:00 AM recipient local."
2. Drag an **Email** action. Configure as above.
3. Smart Sending: OFF.

**Note for the admin:** The body says "Bring a Bluetooth speaker" but Touch 03 of Booking Confirm (Flow 02) says "Bring your own device — connect to our Bluetooth speakers in the gym." These are slightly different. The Bluetooth speakers in the gym are the actual setup — the wording in this Welcome touch is older. If the operator wants consistency, change "Bring a Bluetooth speaker" to "Bring your own device — connect to our Bluetooth speakers in the gym." Cleared with operator before launch.

---

### Touch 04 — Warmth check (SMS)

- **Channel:** SMS
- **Send delay:** Wait 2 days from Touch 03, send at 1pm in recipient's local timezone
- **Sender ID:** `RunRec` (Klaviyo: SMS sending number is the brand's number `+1 (519) 800-9894`; the displayed sender label is whatever you configured under Settings → SMS)
- **Body content (paste exactly):**

```
everything good with the booking process? 👍 or any Q?

-RunRec Stop=opt-out
```

(Note: lowercase intentional. Total under 100 characters including the opt-out line.)

**Klaviyo UI walkthrough — Touch 04:**
1. After Touch 03, drag a **Time Delay** action: "Wait 2 days, send at 1:00 PM recipient local."
2. Drag an **SMS** action onto the canvas.
3. Paste body exactly as above (including the line break before "-RunRec Stop=opt-out").
4. Quiet hours: ON, 8am–9pm recipient local.
5. Smart Sending: OFF.
6. Make sure the recipient has SMS consent — Klaviyo automatically suppresses SMS to non-consented profiles, so most Newsletter signups will skip this touch unless they also gave SMS consent. **This is expected and correct.**

**Reply routing (set up in Klaviyo's SMS Conversations / Inbox or via webhook):**
- 👍 reply → tag profile `welcome_check_positive=true`. No follow-up.
- 👎 reply → escalate to Sal/Izzy email. Trigger private feedback flow if you have one.
- Question → routes to live response inbox (Eyad/Sal monitors).
- STOP → Klaviyo handles the unsubscribe automatically.

---

### Touch 05 — Social proof + self-segment

- **Channel:** Email
- **Send delay:** Wait 2 days from Touch 04, send at 10am ET
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `Which type of player are you?`
- **Preview text:** `4 ways people use this place. Click yours.`
- **Template to use:** Flow / Personal
- **Hero image:** 2×2 grid of four hand-drawn icons (or photo crops) representing each player type: solo trainer · friend group · party host · league runner. Each quadrant clickable. If you don't have hand-drawn icons yet, use stock icons or skip the image and let the text-only links do the work. The text links work either way — image is a nice-to-have.

**Body content:**

```html
<p>RunRec players fall into four camps. Click whichever sounds like you and we'll send you the good stuff:</p>
<p>→ <strong>Solo training:</strong> I'm here to work on my game alone. <a href="REPLACE_WITH_KLAVIYO_UPDATE_LINK_SOLO">[click]</a></p>
<p>→ <strong>Crew player:</strong> I'm always here with my friends. <a href="REPLACE_WITH_KLAVIYO_UPDATE_LINK_CREW">[click]</a></p>
<p>→ <strong>Party host:</strong> I book parties for kids or adults. <a href="REPLACE_WITH_KLAVIYO_UPDATE_LINK_PARTY">[click]</a></p>
<p>→ <strong>League runner:</strong> I run a recurring league or training group. <a href="REPLACE_WITH_KLAVIYO_UPDATE_LINK_LEAGUE">[click]</a></p>
<p>Pick the one that fits and we'll tailor what we send. Pick wrong and you'll get the wrong stuff for a while — no big deal, just click again.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Most people pick "Crew." But the "League runner" group ends up working most closely with us. Just an FYI.</p>
```

**Critical Klaviyo wiring for Touch 05 — the four `[click]` links:**

Each `[click]` link must be a Klaviyo "update profile property" link, NOT a normal link. This is what tags the profile so downstream segmentation works. To set up:

1. In the Klaviyo email editor, click on the `[click]` text for Solo.
2. Choose link type: **"Update profile property"** (Klaviyo calls this an "update_property_link").
3. Property name: `player_type`
4. Value to set: `solo`
5. Where to redirect after click: `https://therunrec.com/book` (or a thank-you page if you have one)
6. Repeat for Crew (`crew`), Party (`party`), League (`league`).
7. Save.

**Why this matters:** Each click both segments the profile AND sends a redirect URL. The profile property `player_type` then powers Lead Nurture, campaigns, and future segmentation. Without this wiring, Touch 05 is just a normal email and the segmentation play dies.

**Klaviyo UI walkthrough — Touch 05:**
1. After Touch 04, drag a **Time Delay** action: "Wait 2 days, send at 10:00 AM recipient local."
2. Drag an **Email** action.
3. Configure From/Subject/Preview as above.
4. Edit content → set up the 4 update-property links per the wiring instructions above.
5. Smart Sending: OFF.

---

### Touch 06 — Behind-the-scenes / tribe

- **Channel:** Email
- **Send delay:** Wait 3 days from Touch 05, send at 10am ET
- **From name:** `RunRec`
- **From email:** `contact@therunrec.com`
- **Reply-to:** same as From
- **Subject line:** `What happens here at 2am.`
- **Preview text:** `A small story from inside the building.`
- **Template to use:** Flow / Personal
- **Hero image:** Security cam aesthetic — slightly grainy, timestamp visible top-right (02:14:08), top-down view of Court A with tiny figures playing. Low-key. Backup: photo of empty court at 2am with light spilling out the front doors.

**Body content:**

```html
<p>Tuesday night, 2:14am.</p>
<p>Our security cam picks up movement on Court A. Five guys, in their 30s, running pickup basketball. Quiet. Focused. Nobody trying to be a hero.</p>
<p>They booked the slot two weeks in advance. They've done this every Tuesday for 6 months.</p>
<p>I asked one of them why this time slot.</p>
<p>He said: <em>"We all have kids and jobs. This is the only hour we can play together. We don't take it for granted."</em></p>
<p>That's what this space was meant for. The 2am slot isn't a fluke — it's the whole point.</p>
<p>If you've got an hour you can never claim back from work or family — book it. The court will be there.</p>
<p style="font-style:italic;">— Sal</p>
<p style="margin-top:24px;padding-top:20px;border-top:1px dashed #999;font-size:14px;"><strong style="color:#e85a3e;">PS</strong> — Some of our most loyal players are the late-night regulars. They book the same slot every week and it becomes their thing.</p>
```

**Klaviyo UI walkthrough — Touch 06:**
1. After Touch 05, drag a **Time Delay**: "Wait 3 days, send at 10:00 AM recipient local."
2. Drag an **Email** action. Configure as above.
3. Smart Sending: OFF.

---

### Touch 07 — Direct rebooking ask · grand slam (SMS)

- **Channel:** SMS
- **Send delay:** Wait 4 days from Touch 06, send at 11am in recipient's local timezone
- **Sender ID:** `RunRec` (same number, `+1 (519) 800-9894`)
- **Body content (paste exactly, with `{{ first_name }}` merge tag):**

```
Hey {{ first_name|default:'there' }} — want to lock in a regular slot? Tuesdays at 9pm is $59 (vs $89) for first-timers. One-tap: runrec.co/T9

-Sal
```

(Total: under 160 characters including signature. Personalized.)

**Klaviyo UI walkthrough — Touch 07:**
1. After Touch 06, drag a **Time Delay**: "Wait 4 days, send at 11:00 AM recipient local."
2. Drag an **SMS** action.
3. Paste body exactly. Make sure the `{{ first_name|default:'there' }}` syntax is intact — Klaviyo will substitute the first name or fall back to "there" if the property is empty.
4. Quiet hours: ON, 8am–9pm.
5. Smart Sending: OFF.

**Important:** This SMS includes a real discount ($59 vs $89). Confirm with operator that this discount mechanic is still active before launch. The Mechanic Doctrine in the master buildsheet says codes are deprecated in favor of reply-to-redeem — but this SMS uses a one-tap link to a pre-built discount URL (`runrec.co/T9`), not a code. As long as that URL routes to a Tuesday-9pm-$59 booking page, this works as intended.

**Audit flag (resolve before launch):** This Day-14 SMS collides with Lead Nurture (Flow 08) Touch 06, also on Day 14. A new Newsletter signup who has never booked may end up in both flows simultaneously and get two competing offers on the same day. To prevent this, the Lead Nurture flow needs a trigger filter that excludes anyone currently in Welcome. See `02-build-template.md` (or the Lead Nurture build guide if it exists) for that wiring. Until that exclusion is in place, **launch Welcome before launching Lead Nurture** — that way at least the conflict only emerges when both are live.

---

## Test plan

Before you set the flow to "Live," do a real test:

1. Create a test profile with your own email (something other than your normal Klaviyo login email) and add it to the Newsletter list.
2. Watch the flow canvas — you should see the profile enter the trigger.
3. Touch 01 should fire within ~5 minutes. Open it on mobile (75% of opens are mobile). Check:
   - Subject line and preview text render correctly
   - All merge tags resolved (no literal `{{ first_name }}` showing)
   - Hero image loads (if you used one)
   - Footer has unsubscribe link, physical address, business name
   - "Drag to Primary" PS reads naturally
4. Klaviyo's flow canvas lets you skip ahead in time for testing — use the "Send next message immediately" button on the test profile to fire Touches 02–07 without waiting 14 days.
5. Touch 04 SMS — make sure your test profile has SMS consent. Verify the lowercase / personal-text feel reads right on iPhone (text bubble should look like a friend, not a brand).
6. Touch 05 — click each of the 4 player_type links. Open the profile in Klaviyo and confirm the `player_type` property is updated to the value you clicked.
7. Touch 07 SMS — verify the personalization (`{{ first_name|default:'there' }}`) renders correctly and the link `runrec.co/T9` actually routes to the Tuesday 9pm discount booking page.
8. After all 7 touches complete, verify the test profile has exited the flow cleanly (Klaviyo flow canvas shows them as "Completed").

---

## Activation checklist

Before you flip the flow to **Live**:

- [ ] All 7 touches built, content reviewed, sent test email/SMS to yourself
- [ ] Footer on every email contains: business name "RunRec," physical address "283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8," functional unsubscribe link, "manage preferences" link
- [ ] Smart Sending: OFF on every touch
- [ ] Quiet Hours: ON for the 2 SMS touches
- [ ] All time delays use "recipient's local timezone" (not UTC)
- [ ] Trigger filter: not in flow in last 90 days, has email consent
- [ ] Touch 05 update-property links wired correctly (test by clicking them — profile should update)
- [ ] No literal merge tags showing (`{{ first_name }}` should always render to a name or fallback)
- [ ] No "TODO," "PLACEHOLDER," or "lorem ipsum" anywhere in the copy
- [ ] Sender domain `contact@therunrec.com` shows green/authenticated in Settings → Domains
- [ ] Operator has approved final copy (especially the Touch 07 SMS discount mechanic)
- [ ] Existing live flow `RZsSVX` is paused (or archived) so new signups don't enter both
- [ ] Lead Nurture exclusion is wired (or Lead Nurture is not yet live)
- [ ] Test profile has been through the full flow successfully

---

## Post-launch monitoring (first 7 days)

Watch these metrics in Klaviyo's flow analytics:

- **Open rate per email:** Target 45%+ on Touch 01 (welcome opens are always highest), 35%+ on Touches 02–06. Below 25% on any touch = subject line problem or deliverability problem.
- **Click rate per email:** Target 3–5%. Touch 05 (segmentation) should see higher clicks (10%+) because the whole email is clickable.
- **Unsubscribe rate per touch:** Should stay under 0.5%. If any single touch exceeds 1%, pause the flow and investigate (usually a copy or relevance problem).
- **Spam complaints:** Should stay under 0.05%. Above 0.1% = pause the flow immediately and investigate.
- **SMS deliverability (Touches 04 + 07):** Check the Klaviyo SMS report. Carrier-block rate should be near zero. STOP rate per send should stay under 1%.

**Warning signs that mean pause:**
- Open rate on Touch 01 below 25% (deliverability problem — likely sender domain authentication failed)
- Unsubscribe rate above 1% on any single touch
- Spam complaints above 0.1% per send
- SMS reply volume to Touch 04 way above expected (could mean carrier mis-routing or copy looks like a phishing attempt)

---

## Common issues for THIS flow

1. **The Newsletter list is single opt-in.** Klaviyo's anti-spam scoring penalizes single-opt-in lists. If deliverability tanks within the first week, this is the most likely cause. Operator should consider switching to double opt-in for new signups going forward.

2. **Touch 04 SMS will silently skip most profiles.** Most Newsletter signups don't give SMS consent, so they will not receive Touch 04. This is expected. The flow continues to Touch 05 normally. Don't be alarmed when the SMS recipient count is much smaller than the email recipient count.

3. **Touch 05 update-property links — the most common build mistake** is to use a regular hyperlink instead of an update-property link. If `player_type` doesn't get set when you click in the test, you wired it as a regular link. Fix per the wiring instructions above.

4. **Touch 07 SMS discount mechanic.** As noted, the master buildsheet now says codes are out — but this SMS uses a one-tap discount URL, not a code, so it's compliant with the doctrine. If the operator changes this to reply-to-redeem instead, update the body to: `Hey {{ first_name|default:'there' }} — want to lock in a regular slot? Reply with your preferred day and time and I'll set you up at $59 (vs $89) for first-timers. -Sal`

5. **Welcome ↔ Lead Nurture Day-14 collision.** Critical. See the audit flag in Touch 07 above. Fix the Lead Nurture trigger exclusion before going live, OR launch this flow first and don't launch Lead Nurture until the exclusion is wired.

6. **Existing flow `RZsSVX` queued sends.** If you don't pause the old Welcome flow before launching this one, profiles already in the old flow's queue will receive both old and new welcome emails. Pause `RZsSVX` BEFORE going live with this one. Klaviyo's "pause" stops new entrants but lets in-flight profiles complete — usually what you want.

7. **The 14-day window is intentionally long.** Some operators want to compress it to 7 days. Don't. The Brunson Soap Opera structure depends on space between touches. Compressed welcome flows underperform every time.

---

## Template specs (if "Flow / Personal" doesn't exist yet)

If your Klaviyo Templates library doesn't have a "Flow / Personal" template, build it once and reuse for every personal-feel email in this and other flows. Specs:

- **Width:** 600px max, single column
- **Background:** white or near-white (#FFFFFF or #FAFAFA)
- **Logo:** Top, centered, 120px wide max
- **Body font:** sans-serif system stack (`-apple-system, BlinkMacSystemFont, 'Inter', 'Helvetica Neue', sans-serif`), 15–16px, line-height 1.6
- **Body color:** #1a1a1a (near-black)
- **Link color:** #e85a3e (RunRec accent orange)
- **PS block:** dashed top border, slightly muted color (#2a2a2a), accent color on the bold "PS" prefix
- **Footer:** light gray background (#ede7db), 11px, contains:
  - Business name: RunRec
  - Physical address: 283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8
  - Unsubscribe link: `{% unsubscribe %}` (Klaviyo merge tag)
  - Manage preferences link: `{% manage_preferences %}` (Klaviyo merge tag)
- **Responsive:** mobile-first, single column collapses cleanly at 480px

Save it once as "Flow / Personal" in Templates. Every Welcome email (and most Lead Nurture, Win-Back, B2B) emails use this same template — only the body content changes.
