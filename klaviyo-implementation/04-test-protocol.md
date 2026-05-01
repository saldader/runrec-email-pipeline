# 04 — Test Protocol

**Audience:** The admin who is going to actually click the buttons in Klaviyo
**Purpose:** Test EVERY flow end-to-end before flipping it from Draft to Live
**When to use this:** After a flow is fully built, before running the [Go-Live Checklist](./05-go-live-checklist.md)
**Estimated time per flow:** 20-60 minutes depending on touch count and branches

---

## Why testing matters

Email and SMS are **irreversible**. Once a message hits a real customer's inbox, you cannot pull it back. There is no "undo."

If a flow has a bug — a broken merge tag, a wrong link, a duplicated touch, a misfiring trigger — and it goes live to the Newsletter list (1,435 profiles), every single one of those people sees the bug at the same time.

The damage from one bad send compounds:

| What goes wrong | What happens next |
|-----------------|-------------------|
| Email shows `{{first_name}}` as raw text instead of "Hassan" | Customers think the brand is amateur. Trust dings. |
| Link in email goes to a 404 page | Click rate craters. Klaviyo flags as low-engagement and deliverability drops. |
| Same SMS goes out twice within 5 minutes | High complaint rate. Carriers may block future sends. |
| Email lands in spam folder | Gmail/Yahoo learn that recipients don't engage with our emails. Future emails to *all* recipients land in spam too. |
| Wrong segment gets the wrong email (e.g., a member gets a "we miss you" win-back) | Customer service complaints. Brand-trust hit. |

A single bad send can reduce open rates from 45% to under 20% for *months*. Recovery requires list cleaning, sender warm-up, and content discipline. It is far cheaper to spend an hour testing than a quarter recovering.

**Rule:** No flow goes from Draft to Live without passing this protocol. No exceptions.

---

## Step 1 — Set up your three test profiles

Before you can test anything, Klaviyo needs profiles that *look like* real customers without being real customers. We use three test profiles that mimic the most common audience archetypes.

### Why three?

Most flows behave differently for different kinds of customers. A welcome flow split for new vs. returning. A win-back flow that exits if the customer books again. A corporate flow with location-manager-specific merge tags. Three profiles let you test all the major branches without spinning up a new identity each time.

### Create them in Klaviyo

Go to **Audience → Profiles → Add Profile** (top-right). Create one at a time:

#### Profile 1 — `test-active@therunrec.com`

Mimics: an active member who books regularly.

| Setting | Value |
|---------|-------|
| Email | `test-active@therunrec.com` |
| First name | `Test` |
| Last name | `Active` |
| Phone (optional, for SMS testing) | `+1` followed by a real number you control |
| Add to lists | Newsletter, Tri-City Member Emails |
| Custom property `last_booking_date` | Today's date |
| Custom property `last_sport_booked` | `Basketball` |
| Custom property `booking_count` | `12` |
| Custom property `total_spend` | `1080` |
| Custom property `membership_tier` | `Tri-City Member` |
| Custom property `membership_status` | `active` |
| Custom property `membership_started_date` | A date 6 months ago |
| Custom property `dob` | A date 30 days from today (so birthday flow can be triggered easily) |
| Custom property `location_name` | `Waterloo` |
| Custom property `location_manager` | `Sal` |
| Custom property `location_manager_email` | `sal` |
| Custom property `location_phone` | `(519) 555-0100` |

#### Profile 2 — `test-lapsed@therunrec.com`

Mimics: a customer who used to book but hasn't been back in 60+ days.

| Setting | Value |
|---------|-------|
| Email | `test-lapsed@therunrec.com` |
| First name | `Test` |
| Last name | `Lapsed` |
| Phone (optional) | A second real number you control, or leave blank |
| Add to lists | Newsletter, Skedda Email List |
| Custom property `last_booking_date` | A date 75 days ago |
| Custom property `last_sport_booked` | `Pickleball` |
| Custom property `booking_count` | `3` |
| Custom property `total_spend` | `270` |
| Custom property `membership_tier` | `Common User` |
| Custom property `dob` | Any date in the past |
| Custom property `location_name` | `Waterloo` |
| Custom property `location_manager` | `Sal` |
| Custom property `location_manager_email` | `sal` |

#### Profile 3 — `test-corporate@therunrec.com`

Mimics: a B2B lead who submitted the corporate inquiry form.

| Setting | Value |
|---------|-------|
| Email | `test-corporate@therunrec.com` |
| First name | `Test` |
| Last name | `Corporate` |
| Add to lists | (leave empty — will be added by the corporate inquiry event) |
| Custom property `corporate_inquiry_date` | Today's date |
| Custom property `corporate_status` | `active` |
| Custom property `corporate_company_name` | `Test Co` |
| Custom property `corporate_event_size` | `25` |
| Custom property `location_name` | `Waterloo` |
| Custom property `location_manager` | `Sal` |
| Custom property `location_manager_email` | `sal` |
| Custom property `location_phone` | `(519) 555-0100` |

### Practical setup tips

- Use email addresses you actually control. The two simplest options: forward `test-active@therunrec.com` to your real inbox (set up in your domain DNS), or use Gmail's plus-addressing trick (`yourname+test-active@gmail.com` all go to `yourname@gmail.com`). The plus trick is faster.
- Whatever you use, **the email must reach an inbox you can read.** You're going to open every test send.
- Tag all three profiles with a tag called `internal-test`. This lets you exclude them from analytics later.
- Never delete these profiles. Reuse them for every flow test going forward.

---

## Step 2 — Per-flow test protocol (universal template)

Run every step below for every flow. Even if you've tested a similar flow before — different flows behave differently, and the cost of skipping one step is high.

### The seven-step pattern

Print this. Tape it next to your monitor. Every flow goes through these seven steps.

| Step | What you do | What you're checking |
|------|-------------|----------------------|
| 1 | Set up the test profile to match the flow's trigger | The profile satisfies all entry conditions |
| 2 | Manually trigger the flow on the test profile | The flow actually starts |
| 3 | Verify each touch fires at the right delay | Timing is correct between touches |
| 4 | Open every email — verify subject, preview, body, from-line | Content renders correctly |
| 5 | Click every link in every email | Links go where they should |
| 6 | For SMS, verify message arrives + STOP keyword works | SMS infrastructure is healthy |
| 7 | For conditional logic, test BOTH branches | Branching works for both yes and no paths |

---

### Step 2.1 — Match the trigger

Open the flow in Klaviyo. Check the trigger box at the top.

| Trigger type | What you do to match it on a test profile |
|--------------|-------------------------------------------|
| Added to List | Manually add the test profile to the list (Audience → Lists → click list → Add Members) |
| Metric / Event | Use the Klaviyo Track API (or have Sal/engineer trigger the event), OR use the "Manually trigger this flow" option at the top of the flow editor |
| Date-based (e.g., birthday) | Set the `dob` property on the test profile to a date in the near future, then use Manually Trigger |
| Profile property change | Edit the test profile's property to the trigger value |

If you cannot trigger the event yourself, **stop and ask Sal**. Some events come from outside Klaviyo (Skedda, Stripe). Sal will either trigger them on the test account or have engineering simulate them.

---

### Step 2.2 — Manually trigger and confirm entry

Klaviyo gives you a way to force a profile into a flow. Use it.

1. Open the flow in **Flow Editor** view
2. Top-right, click **Manage Flow → Manually trigger flow**
3. Search for the test profile by email
4. Click **Trigger**
5. Wait 30 seconds
6. Open the test profile (Audience → Profiles → search by email → click profile)
7. Click the **Activity Feed** tab
8. Look for an entry that says "Entered flow [Flow Name]"

If the profile didn't enter, check:
- Is the profile already in the flow from a prior trigger? (It can't enter twice unless the flow allows re-entry.)
- Does the profile match the entry filters? (E.g., if the flow filters out unsubscribed profiles and your test profile is unsubscribed.)
- Is the flow set to Live or Draft? (Manually triggered flows still respect Live/Draft status — the email won't actually send if Draft, but the entry should still register.)

Refer to [07-troubleshooting.md](./07-troubleshooting.md) for "My test profile didn't enter the flow" if stuck.

---

### Step 2.3 — Verify each touch fires at the right delay

The flow has multiple touches separated by delays (e.g., wait 24 hours, wait 2 days). Real-time waiting for a 14-day welcome flow is impossible — Klaviyo gives you ways to fast-forward.

Two options:

**Option A — Use the Flow Simulator (preview mode).** This is the fastest. It walks through the flow showing each touch in sequence with no real waiting and no real send.

**Option B — Use a "send time override."** When you manually trigger a flow, Klaviyo lets you choose to skip delays. Each touch fires immediately for the test profile. Use this when you want to actually receive the emails in your inbox to inspect them.

For most flows, do BOTH:
1. First pass: Flow Simulator — confirms the structure (delays, branches, send-or-skip logic) is correct
2. Second pass: Skip delays + send to test profile — confirms the actual emails render correctly in real inboxes

---

### Step 2.4 — Open every email and inspect

For every email touch in the flow, open it in your inbox and check:

| What to check | How to check |
|---------------|--------------|
| Subject line | Matches the dashboard. No raw merge tags (`{{first_name}}`). No "RE:" or "FW:" prefix. |
| Preview text | Shows in the inbox preview. Doesn't repeat the subject. Under 90 characters before truncation. |
| From-name + from-email | Matches what the spec says (e.g., "The RunRec <contact@therunrec.com>" for consumer flows; `{{location_manager}}` for B2B) |
| Body copy | Matches the dashboard exactly. No raw merge tags. Personalization shows the test profile's data. |
| Logo | Renders. Not a broken-image placeholder. |
| Footer | Shows business name (The RunRec), physical address (283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8), and unsubscribe link |
| Mobile rendering | Open the email on your phone. Does it look right? Text readable? Buttons tappable? |

For each test email, check this in **two clients minimum**: Gmail (web) and your phone's default mail app. If you have access to Outlook or Apple Mail, check those too — they render differently.

---

### Step 2.5 — Click every link

Every link in every email gets clicked. Yes, every one. This is the step most people skip and the step that catches the most bugs.

For each link:
1. Click it from the email
2. Confirm it loads (no 404, no "site can't be reached")
3. Confirm it lands on the right page (not a homepage when it should be a specific product page)
4. Confirm the page works (not just loads — the booking button on a booking page should actually book)

Also check:
- Unsubscribe link works (clicking it brings up Klaviyo's unsubscribe flow, then re-subscribes you for further testing)
- Social links (if any) go to the right profile
- Any embedded image with a link click-through has the link wired

---

### Step 2.6 — SMS test

If the flow has SMS touches, do the following on the phone you registered to the test profile:

1. Confirm the SMS arrives within 1-2 minutes of the trigger
2. Confirm the sender ID is correct ("RunRec" or whatever the spec says)
3. Confirm the message text matches the dashboard exactly
4. Confirm any link in the message works when tapped
5. Reply with `STOP` (yes, really) and confirm:
   - You receive a confirmation message ("You have been unsubscribed...")
   - The test profile's SMS subscription status flips to "Unsubscribed" in Klaviyo
6. To re-test, log into Klaviyo, find the test profile, and re-subscribe to SMS manually before the next test

If the SMS doesn't arrive within 5 minutes, see [07-troubleshooting.md](./07-troubleshooting.md) "SMS didn't deliver."

---

### Step 2.7 — Test BOTH branches of conditional logic

Many flows have conditional splits. Examples:
- Welcome flow: split by purchase status (booker vs. non-booker)
- Post-session flow: split by NPS thumbs-up vs. thumbs-down
- Win-back flow: split by reply YES vs. no response

For every conditional split in the flow, test BOTH paths.

Example for the post-session flow's NPS split:
1. First test pass: trigger the flow on `test-active@therunrec.com` and simulate a thumbs-up response. Verify the positive-path emails fire.
2. Second test pass: trigger the flow again (or use a fresh test profile) and simulate a thumbs-down response. Verify the negative-path emails fire.

If you can only easily test one branch, **do not skip the other**. Use Klaviyo's Flow Simulator to walk through the alternate branch even if you can't easily simulate the trigger condition. At minimum confirm that the alternate branch's emails are configured correctly (subject, body, from-line, links) even if you can't trigger them in real-time.

---

## Step 3 — Klaviyo's built-in tools

Three tools you should learn to use fluently. They make testing 10x faster.

### 3.1 — Send Test (per email)

In any email editor in Klaviyo, top-right corner has a **Send Test** button.

- Click it
- Enter your real email address
- Klaviyo sends an immediately-rendered preview to your inbox
- This bypasses the flow entirely — it's just a render check

Use Send Test when you want to inspect a single email's render, links, or merge tags without setting up the whole flow trigger.

**Caveat:** Send Test uses *placeholder* values for merge tags (e.g., `{{first_name}}` becomes "Friend" or empty). To test with real data, you must trigger the actual flow on a real test profile. Send Test is a quick first-pass; trigger-on-test-profile is the full pass.

### 3.2 — Flow Preview / Flow Simulator

Open the flow in the editor. Top-right has **Preview** or **Simulate**. (Klaviyo has renamed this a couple of times.)

- Choose a real profile (or your test profile) to simulate the flow on
- Klaviyo walks you through every touch, showing the rendered email/SMS at each step
- You can step forward through delays without waiting
- Conditional branches are highlighted — Klaviyo tells you which path the test profile would take

Use Flow Simulator before sending real test emails. It's faster and catches structural problems (wrong branch, missing delay, broken merge tag) without filling your inbox.

### 3.3 — Profile Activity Timeline

Open any profile (Audience → Profiles → search → click). Click the **Activity Feed** tab.

This shows everything Klaviyo knows about that profile in chronological order:
- Entered/exited flows
- Received email/SMS
- Opened email
- Clicked link
- Bounced
- Unsubscribed

After triggering a test, refresh this timeline to confirm what actually happened. If you triggered a flow but the timeline doesn't show "Entered flow," the trigger didn't fire. If the timeline shows "Skipped due to filter," your filters are excluding the profile.

This is your single best diagnostic tool. When something looks wrong, check the timeline first.

### 3.4 — Email Render Checker (optional, if available)

Some Klaviyo plans include an "email rendering" check (powered by Litmus or similar). It shows how an email renders across 30+ clients (Outlook 2007, dark-mode iOS, etc.).

If you have this feature, run it on every email before activation. If not, manual two-client check (Gmail + phone) is the minimum.

---

## Step 4 — Common test failures and what they mean

| Symptom | Most likely cause | Quick fix |
|---------|------------------|-----------|
| "Test profile didn't enter the flow" | Trigger not configured to match what you did. E.g., flow trigger is "Added to Newsletter list" but you added to a different list. | Re-read the flow's trigger box. Match it exactly. |
| "Email arrived but `{{first_name}}` shows as raw text" | The property doesn't exist on the test profile, or the merge-tag syntax is wrong (e.g., `{{ first name }}` with a space) | Open the test profile, set `first_name`. Check the email's source for typos in merge-tag syntax. |
| "Email landed in spam folder" | Sender domain authentication (DKIM/SPF/DMARC) is broken or missing | Escalate to Sal — DNS-level fix needed |
| "SMS arrived but the link didn't work" | URL shortener glitch, or the link domain isn't whitelisted | Try the link directly in a browser. If it works there, it's a shortener issue — escalate. |
| "Two test emails arrived for the same touch" | Smart Sending is off, or the flow re-triggered the profile | Turn Smart Sending on (Settings → Account → Smart Sending). Check whether the test was triggered twice. |
| "The link in the email goes to the wrong page" | Copy-paste error in the email body. Or a UTM parameter broke the URL. | Edit the email, replace the link, re-send test. |
| "Email looks fine on desktop, broken on mobile" | Image too wide, font too small, padding wrong | Edit the email's mobile preview in Klaviyo. Adjust column widths. |
| "Klaviyo says 'Skipped due to filter'" | Profile didn't pass an in-flow filter (often the engagement filter "Has not been sent in last 16 hours") | Wait 16+ hours, OR temporarily relax the filter for testing |

For more, see [07-troubleshooting.md](./07-troubleshooting.md).

---

## Step 5 — Test sign-off form

Before you mark a flow as "tested" and move to the [Go-Live Checklist](./05-go-live-checklist.md), fill out this form. Keep a copy in the flow's notes (Klaviyo's flow editor has a Notes panel) and one in your local file.

```
FLOW TEST SIGN-OFF
==================

Flow name: ____________________________________
Flow ID (Klaviyo): _______________________________
Tested by: ____________________________________
Date tested: __________________________________

TRIGGER
[ ] Trigger fires on test profile
[ ] Activity Timeline shows "Entered flow"

TOUCHES
[ ] All touches fire at the correct delay (verified in Flow Simulator)
[ ] Total touch count matches spec: ____ touches
[ ] No duplicate sends to the same profile within 16 hours

EMAIL CONTENT (per touch)
For each email touch, check:
[ ] Subject line matches dashboard
[ ] Preview text matches dashboard
[ ] From-name and from-email correct
[ ] Body renders cleanly on desktop (Gmail web)
[ ] Body renders cleanly on mobile (phone default mail app)
[ ] All merge tags resolve to actual data (no raw `{{first_name}}`)
[ ] All links click through to correct pages
[ ] Logo renders
[ ] Footer shows: business name + physical address + unsubscribe link

SMS CONTENT (per touch, if any)
[ ] SMS arrives within 5 minutes
[ ] Message text matches dashboard
[ ] Sender ID correct
[ ] Link tappable and lands on correct page
[ ] STOP keyword works (replied, received confirmation)

CONDITIONAL LOGIC
[ ] All branches identified: ___ branches total
[ ] Each branch tested separately
[ ] Conditional split routes profiles correctly

EXIT CONDITIONS
[ ] Profile exits the flow when expected (e.g., books again, unsubscribes)
[ ] Profile does NOT re-enter the flow after exiting unless flow allows re-entry

OPERATOR APPROVAL REQUIRED
[ ] Reviewed by Sal (operator)
    Sal's sign-off (initials + date): _______________

NOTES / KNOWN ISSUES (anything not perfect but accepted by operator)
_______________________________________________
_______________________________________________

READY FOR GO-LIVE: [ ] YES   [ ] NO

If NO, what needs to happen first:
_______________________________________________
```

**Do not move to Go-Live until this form is signed off by Sal.** Sal's approval is the gate. Even if every checkbox is green, no activation without the operator's initials.

---

## Final reminder

Testing is the most boring, most repetitive, most important step in this entire package. The temptation to skip a step grows with every flow you test. Don't.

Every step exists because somebody, at some point, paid for skipping it.

Move slow. Open every email. Click every link. Test both branches. Get Sal's signature. Then move on.

Next: [05-go-live-checklist.md](./05-go-live-checklist.md)
