# 05 — Go-Live Checklist

**Audience:** The admin who is about to flip a flow from Draft to Live
**Purpose:** Final cross-check before activation. Catches the small stuff a per-touch test can miss.
**When to use this:** After [04-test-protocol.md](./04-test-protocol.md) is complete and Sal has signed off
**Estimated time:** 30-45 minutes per flow

---

## Why this checklist exists separately from the test protocol

The test protocol checks that the flow *works as built*. The go-live checklist checks that the flow *is safe to expose to real customers right now*.

A flow can pass testing perfectly and still be unsafe to launch:

- The sender domain authentication broke yesterday
- The list it triggers from has 5,000 unconfirmed contacts you didn't notice
- Another live flow is already firing on the same profiles today
- The flow's "From" address points to a mailbox no one monitors

The test protocol is about content correctness. The go-live checklist is about *operational readiness*. Two different concerns. Both required.

---

## Section 1 — Pre-flight re-verification

These are things you confirmed in [01-pre-flight-setup.md](./01-pre-flight-setup.md), but they can drift. Re-check them right before activation.

| Check | Where to verify | Pass criterion |
|-------|----------------|----------------|
| Sender domain authentication | Settings → Domains → Sending Domains | DKIM, SPF, DMARC all show green checks |
| Branded sending domain (e.g., `send.therunrec.com`) | Same screen | Status: "Successfully Authenticated" |
| Sender Profile (from-name + from-email) | Settings → Email → Sender Profiles | The profile this flow uses exists and is verified |
| Reply-to mailbox is monitored | Manual check — open the inbox | Someone actually reads `contact@therunrec.com` (or the relevant mailbox) |
| All referenced segments exist | Audience → Segments | Every segment named in the flow's filters exists and has a profile count > 0 (unless the flow is intentionally for a small audience) |
| All referenced lists exist | Audience → Lists | Same as above |
| All custom profile properties exist | Settings → Profile Properties | Every `{{...}}` merge tag in the flow has a matching property |
| Smart Sending is ON globally | Settings → Account → Smart Sending | Toggle is on; thresholds are 16h email, 12h SMS |
| Quiet Hours configured | Settings → Account → Quiet Hours | 9 PM to 9 AM in recipient's local timezone |
| Unsubscribe page is live | Click the unsubscribe link in any test email | Lands on Klaviyo's hosted unsubscribe page or your custom one — not a 404 |

If any of these are broken, **stop**. Do not activate the flow. Fix the underlying issue first. If you don't know how to fix one, escalate to Sal.

---

## Section 2 — Flow-specific verification

Open the flow in Klaviyo. Walk through every touch in sequence. For each touch:

### Email touches

| Check | Pass criterion |
|-------|---------------|
| Subject line | Matches the dashboard at https://saldader.github.io/runrec-email-pipeline/. No typos. No "TEST" or placeholder text. |
| Preview text | Set (not empty). Doesn't repeat the subject. Under 90 characters before truncation. |
| From-name | Correct per the voice doctrine (e.g., "The RunRec Team," "Sal Dader," "{{location_manager}}") |
| From-email | Correct per the spec (e.g., `contact@therunrec.com` for consumer, `{{location_manager}}@therunrec.com` for B2B) |
| Reply-to | Set to a monitored mailbox |
| Body — desktop render | Open Klaviyo's preview tool. Looks clean. No broken images. No overflow. |
| Body — mobile render | Same preview tool, mobile mode. Text readable, buttons tappable, no horizontal scroll. |
| All merge tags resolve | Use Preview with a real test profile. No raw `{{first_name}}` text. |
| All links work | Click each one in the preview. No 404s. No suspicious URL shorteners. |
| Unsubscribe link | Visible in footer. Working. Goes to the Klaviyo unsubscribe page. |
| Physical address | Visible in footer: 283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8 |
| Business name | "The RunRec" present in footer |
| Logo | Renders. Not pixelated. Has alt text. |
| No spam-trigger words in subject | Avoid "$", "FREE", "WINNER", "ACT NOW", "LIMITED TIME", all-caps subjects |

### SMS touches

| Check | Pass criterion |
|-------|---------------|
| Message text | Matches dashboard. Under 160 characters (or planned to fit one segment). |
| Sender ID | "RunRec" or per spec |
| Link | Tappable. Lands on correct page. |
| Opt-out language | First SMS in the flow includes "Reply STOP to opt out" — required for CASL compliance |
| Quiet hours respected | Klaviyo's quiet-hours setting will hold sends until 9 AM local — confirm this is on |

---

## Section 3 — Trigger verification (with a fresh test profile)

Even though you tested the trigger in [04-test-protocol.md](./04-test-protocol.md), do it once more *right before* activation. Things change.

1. Create or reset a test profile
2. Match the trigger condition
3. Wait up to 5 minutes
4. Check the test profile's Activity Timeline
5. Confirm: "Entered flow [Flow Name]" entry appears

If it doesn't appear, **stop**. Re-read the trigger. Check the entry filters. Don't activate until the trigger fires reliably.

---

## Section 4 — Filter verification

Most flows have entry filters that exclude certain profiles (e.g., "exclude unsubscribed," "exclude profiles already in this flow," "exclude profiles in the corporate-only segment").

Test that filters work:

1. Pick a profile that *should* be excluded (e.g., an unsubscribed profile)
2. Try to trigger the flow on them
3. Confirm: their Activity Timeline shows "Skipped — did not match flow filters"

If a profile that should be excluded enters the flow anyway, **stop**. Filter is broken. Fix before activation.

Common filters to verify:

| Filter | What it does | Why it matters |
|--------|--------------|---------------|
| "Has consented to email marketing" | Only sends to people who opted in | Required for CASL compliance |
| "Is not suppressed" | Skips suppressed profiles (bounced, complained) | Protects sender reputation |
| "Has not been sent in last 16 hours" | Smart Sending — prevents over-mailing | Prevents complaints |
| "Is not currently in this flow" | Stops re-entry within the same flow | Prevents loops |
| Flow-specific (e.g., "is in Tri-City Members list") | Restricts audience to the right segment | Prevents wrong-audience sends |

---

## Section 5 — Exit verification

Most flows have exit conditions: "If they book again, exit the flow." "If they unsubscribe, exit." "If they reply YES, exit and enter the booking confirm flow."

For each exit condition:

1. Identify it on the flow diagram
2. Trigger the exit condition on a test profile mid-flow
3. Confirm: their Activity Timeline shows "Exited flow"
4. Confirm: they don't receive any subsequent touches in the flow

If a profile that should have exited continues to receive touches, **stop**. Exit logic is broken.

---

## Section 6 — Conflict check (the one most-skipped step)

This is the step that prevents customers from getting two emails on the same day from two different flows.

Before activating any new flow, check what other live flows could fire on the same profile in the same 24 hours.

### How to check

1. Open Audience → Lists/Segments
2. Identify the audience for this new flow (e.g., "Newsletter list" for the Welcome flow)
3. List every other LIVE flow that could trigger on the same audience in a 24-hour window
4. Check the touch timing of both flows
5. If two flows would send touches within 24 hours of each other, you have a conflict

### Example conflicts to watch for

| New flow being launched | Could conflict with | Why |
|-------------------------|---------------------|-----|
| Welcome (extends current) | Lead Nurture | Both fire on new email signups; both have Day-14 closing touches with different offers |
| Browse Abandoned | Abandoned Booking | Both fire on intent signals from the same profile within hours |
| Win-Back | Sunset / Hygiene | Both target lapsed profiles |
| Customer Birthday | Birthday Party Anniversary | Both could fire on the same profile in the same week |
| Post-Session + Review | Booking Confirm | Same profile within 24h of a booking |

### What to do if you find a conflict

Three options, in order of preference:

1. **Add a flow exclusion filter.** In the new flow, add a filter: "Profile is not currently in [other flow]." This prevents the conflict at the entry point.
2. **Stagger send times.** Adjust the new flow's delays so its touches don't fall on the same day as the other flow's touches.
3. **Pause the older flow.** Last resort. Only if the new flow fully replaces the old one.

**Document the conflict resolution in the flow's Notes panel.** Future you will thank present you when something seems off in 6 months.

---

## Section 7 — Final approval gate

Klaviyo lets you flip Draft to Live with one click. The whole point of this checklist is to make sure that one click is the *last* thing that happens, not the first.

Before clicking Live:

1. Re-read the [test sign-off form](./04-test-protocol.md#step-5--test-sign-off-form) for this flow. All boxes green?
2. Re-read this entire go-live checklist. All boxes green?
3. **Get Sal's explicit "yes, go live" in writing.** Slack message, email, text — anything you can save. Do not activate based on a verbal "I think we're good." Words on a screen are the only acceptable approval.
4. Confirm Sal has tested the actual emails himself (sent test sends to Sal's inbox at minimum, ideally a small subset of profiles)
5. Confirm there's no major planned campaign sending in the next 4 hours (collision risk)
6. Confirm someone is available to monitor for the next 6 hours (you, Sal, or both)

If any of those are missing — even if just the "available to monitor" — wait until tomorrow.

---

## Section 8 — Activation sequence (the click path)

Once everything is approved, here's the exact click path to activate.

1. Open the flow in **Flow Editor** view
2. Top-right corner, click **Manage Flow**
3. Click **Update flow status**
4. A modal appears with the current status (Draft) and a dropdown
5. Change the dropdown to **Live**
6. Klaviyo will show a warning: "Are you sure? This will start sending to real customers."
7. Read it. Acknowledge it. Click **Save**
8. The flow's status indicator (top-left) flips from Draft (grey) to Live (green)
9. Note the activation time. Write it down. You'll reference it during monitoring.

That's it. The flow is now live. Real customers will start entering as triggers fire.

---

## Section 9 — First 24 hours monitoring

Once a flow is live, do not walk away. The first 24 hours are when problems show up.

### What to check

| Metric | When to check | Healthy range | Red flag |
|--------|--------------|---------------|----------|
| Profiles entered | Hourly for first 6 hours | Climbing as triggers fire | Zero entries — trigger may be broken |
| Open rate (first touch) | After 4-6 hours | 30%+ for first send | Below 15% — possible deliverability problem |
| Click rate | After 6-12 hours | 2%+ | Below 0.5% — copy or link problem |
| Bounce rate | After 1-2 hours | Below 2% | Above 3% — list quality problem |
| Unsubscribe rate | First 24 hours | Below 0.5% | Above 1% — frequency or relevance problem |
| Spam complaints | First 24 hours | Below 0.05% | Above 0.1% — pause immediately |
| Reply volume to "reply-to-redeem" CTAs | Continuous | Some replies coming in | Zero replies after 100+ sends — CTA may be unclear |
| Customer service tickets mentioning the flow | Continuous (check inbox) | None or one-off | More than 3 — pause and investigate |

### Where to find these numbers in Klaviyo

- **Profiles entered:** Open the flow → Analytics tab → "People in flow" widget
- **Open rate, click rate:** Same Analytics tab → per-touch metrics
- **Bounce/unsubscribe/complaint:** Same Analytics tab → "Negative metrics" section
- **Replies:** Check the reply-to inbox manually (the customer's reply doesn't show in Klaviyo)

### What to do if something looks bad

| Red flag | Immediate action |
|----------|-----------------|
| Bounce rate above 3% | Pause flow. Check the trigger list — likely contains old/dead addresses. Run list cleaning before re-activating. |
| Open rate below 15% | Pause flow. Check sender domain auth. Check subject line for spam triggers. |
| Spam complaints above 0.1% | Pause flow IMMEDIATELY. Investigate which touch is driving complaints. Likely too aggressive timing or wrong audience. |
| Customer reports getting an unexpected email | Pause flow. Investigate which audience entered. Likely a filter is broken. |
| Klaviyo shows an error notification | Pause flow. Read the error. Escalate to Sal. |

---

## Section 10 — Rollback procedure

Sometimes you have to pull a flow back. Here's how, in order of severity.

### Level 1 — Pause the flow (no new entries)

This stops new profiles from entering, but profiles already in the flow continue receiving their remaining touches.

1. Open the flow → Manage Flow → Update flow status
2. Change to **Draft** (this pauses entry)
3. Profiles currently in the flow continue on their existing path

Use this when: Something is wrong with the trigger but the existing in-flight emails are fine.

### Level 2 — Pause the flow AND halt in-flight profiles

This stops new entries AND removes everyone currently in the flow before they get more touches.

1. Open the flow → Manage Flow → Update flow status → Draft (pauses entry)
2. Open the flow → Action menu → "Manually exit profiles from flow"
3. Select "All profiles currently in flow" → Exit
4. Klaviyo will remove every in-flight profile. They will not get any more touches from this flow.

Use this when: A specific touch is broken or wrong, and people in flight will get the bad touch next.

### Level 3 — Emergency stop (rare)

This is for "we sent the wrong email to the wrong audience" situations.

1. Pause the flow (Level 1 + 2 above)
2. Identify the audience that received the bad send (Klaviyo → Campaigns/Flows → Recipients tab)
3. Determine the impact (how many people, what they got)
4. Notify Sal immediately
5. Sal decides whether to send an apology email or let it stand
6. Do a postmortem within 24 hours: what went wrong, why testing missed it, what you'll change

Use this when: Customers are receiving content that shouldn't have gone to them. This is a brand crisis. Move fast and keep Sal informed.

### After any rollback

- Update the flow's Notes panel with what happened, when, and what you did
- Do not re-activate until the underlying issue is identified and fixed
- Re-run [04-test-protocol.md](./04-test-protocol.md) end-to-end before re-activating
- Get Sal's explicit re-approval

---

## Final reminder

Activating a flow feels small. One click. The implications are not small. The same click that activates a clean flow also activates a broken one — Klaviyo doesn't know the difference.

The point of this checklist is to make the click feel deliberate. Slow it down. Read every check. Get the operator's signature. Then click.

When in doubt, wait until tomorrow. A delayed launch costs hours. A bad launch costs months.

Next: [06-compliance.md](./06-compliance.md) for legal and best-practice context, or [07-troubleshooting.md](./07-troubleshooting.md) when things go wrong.
