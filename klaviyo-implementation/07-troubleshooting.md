# 07 — Troubleshooting

**Audience:** The admin who is staring at a Klaviyo screen wondering why something doesn't look right
**Purpose:** Symptom → cause → fix. Searchable by what you're seeing.
**When to use this:** Any time something looks off. Search by the quoted symptom.

> **Use this in conjunction with [04-test-protocol.md](./04-test-protocol.md) and [05-go-live-checklist.md](./05-go-live-checklist.md).** Most symptoms below are caught earlier if those checklists are followed.

---

## How to use this document

Find the quoted symptom that matches what you're seeing. Read:

1. **Likely causes** — ranked by frequency (most common first)
2. **How to diagnose** — step-by-step to find which cause is yours
3. **Fix** — what to change
4. **Prevention** — how to avoid this next time

If your symptom isn't here, escalate to Sal with: a screenshot, the flow name, the profile email (if relevant), and the timestamp.

---

## Symptom 1: "My test profile didn't enter the flow"

**Likely causes (ranked):**

1. The trigger condition wasn't actually met (most common)
2. The profile was excluded by a flow filter
3. The profile is already in the flow and re-entry is blocked
4. The profile is in the suppression list
5. Smart Sending blocked the trigger
6. The flow is in Draft mode (manual triggers still register but actual sends don't fire)

**How to diagnose:**

1. Open the test profile (Audience → Profiles → search by email)
2. Click the **Activity Feed** tab
3. Look for the most recent entry related to the flow

What you might see:

| Activity Feed entry | Meaning |
|---------------------|---------|
| Nothing related to this flow | Trigger never fired. Cause #1. |
| "Skipped — did not match flow filters" | Cause #2. The next line tells you which filter excluded the profile. |
| "Skipped — profile already in flow" | Cause #3. Profile is mid-flow from a previous trigger. |
| "Skipped — profile is suppressed" | Cause #4. Profile is on the suppression list. |
| "Skipped — Smart Sending threshold" | Cause #5. Profile received another message recently. |
| "Entered flow" but no message arrives | Cause #6, OR the first touch has a delay > 0 |

**Fix per cause:**

- Cause #1: Re-read the trigger box on the flow editor. Is the trigger "Added to Newsletter list" but you added the profile to "Skedda Email List"? Match exactly. For Metric triggers, confirm the metric name and any filter (e.g., "where item_value > 50").
- Cause #2: Open the flow → Trigger box → see the filters. Compare each filter against your test profile's properties. Adjust the test profile to match, or temporarily relax the filter for testing.
- Cause #3: Either wait for the profile to exit the flow naturally, or manually exit them (Profile → Actions → "Exit profile from flow"). Then re-trigger.
- Cause #4: Check Settings → Suppressions. If the test profile is there, it's suppressed permanently for marketing. Use a different test profile, or unsuppress (Settings → Suppressions → Remove — but only for test profiles, never real customers).
- Cause #5: Wait 16+ hours and try again. Or, for testing, temporarily disable Smart Sending (Settings → Account → Smart Sending → Off) — but turn it back on immediately after testing.
- Cause #6: Confirm the flow is Live. If you want to test in Draft, the manual trigger will still register entries but the email/SMS won't actually send. Use Send Test on the individual touch instead.

**Prevention:** Always check the trigger condition word-for-word against the flow editor before testing. Use a fresh test profile (not a profile that may already be in the flow).

---

## Symptom 2: "The email shows `{{first_name}}` instead of the actual name"

**Likely causes:**

1. The property doesn't exist on the recipient's profile (most common)
2. Merge tag syntax is broken (typo, wrong braces, extra spaces)
3. No fallback configured for missing data

**How to diagnose:**

1. Open the recipient's profile in Klaviyo
2. Scroll to **Properties** section
3. Look for `first_name` (or whatever the missing tag was)

What you might see:

- Property doesn't exist or is blank → cause #1
- Property exists with the right value → cause #2 (open the email source and check the merge tag syntax)
- Property exists but the email rendered the literal `{{first_name}}` text → cause #3 (no fallback)

**Fix per cause:**

- Cause #1: Set the property on the profile. For new signups, ensure your signup form collects first name (or update the Klaviyo Form to make it required). For imported profiles, do a CSV import with first names. Going forward, every flow that uses `{{first_name}}` should also have a fallback.
- Cause #2: Klaviyo's merge tag syntax is `{{ first_name|default:'there' }}`. Common typos: `{{first name}}` (space inside the braces), `{first_name}` (single braces), `{{first_name }` (mismatched braces). Open the email's HTML view (top-right of the editor → "Source") and search for the tag.
- Cause #3: Add a fallback to every personalization tag. Standard pattern: `{{ first_name|default:'there' }}`. For "Hi {{ first_name }}" emails, the fallback becomes "Hi there" — natural-sounding. For "{{ location_name }} regulars," the fallback might be "RunRec regulars."

**Prevention:** Every merge tag in every email should have a fallback. This is a copy-and-paste habit: every time you type `{{`, follow it with `|default:'something'`. Use this template for the most common tags:

| Tag | Fallback |
|-----|---------|
| `first_name` | `there` |
| `last_name` | (leave blank) |
| `location_name` | `RunRec` |
| `location_manager` | `the team` |
| `last_sport_booked` | `your sport` |
| `child_name` | `your kid` |

---

## Symptom 3: "Open rate is 5%"

This is a deliverability problem. The emails are reaching inboxes (or spam folders) but recipients aren't opening them. Or worse, they're not reaching inboxes at all.

**Likely causes (ranked by impact):**

1. Sender domain authentication (DKIM/SPF/DMARC) is failing or missing (most catastrophic)
2. Sender reputation is damaged (sending to dead addresses, high bounce rate, recent spam complaints)
3. Subject lines are spam-trigger-heavy
4. Sender domain is the generic `klaviyomail.com` instead of branded
5. List hygiene is poor (many addresses are dormant)
6. Recipients are real but the audience is wrong (sending to a list that didn't opt in for this content)

**How to diagnose:**

1. Check Settings → Domains → Sending Domains. All three (DKIM, SPF, DMARC) should be green. If yellow or red, that's cause #1.
2. Look at the past 30 days of campaigns. What's the bounce rate? Above 2-3% indicates sender reputation damage (cause #2). Above 5% means deliverability is actively dropping.
3. Pull the past 5 subject lines. Do any contain "$", "FREE", "WINNER", excessive caps, or emojis at the start? That's cause #3.
4. Check Settings → Email → Sender Profiles. Does your sender domain end in `klaviyomail.com` or `therunrec.com`? If `klaviyomail.com`, that's cause #4.
5. Open Audience → Lists/Segments. What percentage of your active list has opened an email in the last 90 days? If under 30%, that's cause #5.
6. Open the failing campaign/flow. Who did it send to? Was that audience appropriate for the content?

**Fix per cause:**

- Cause #1: This is engineering-level — DNS records need to be set on therunrec.com pointing to Klaviyo's verification keys. **Escalate to Sal and Rob (engineer) immediately.** Do not send any marketing until fixed. This is the biggest impact item in the document.
- Cause #2: Stop sending immediately. Run a sunset flow (Flow 13) to remove unengaged profiles. Wait 7-14 days for sender reputation to recover before resuming high-volume sends. Consider a slow re-warm: send only to most-engaged 20% for a week, then 40%, then 60%.
- Cause #3: Rewrite subject lines per the email-marketing skill's guidance: under 50 characters, curiosity-driven, no money words, no all-caps.
- Cause #4: Set up the branded sending domain. Settings → Domains → Add new sending domain. Use `send.therunrec.com` or similar. Requires DNS access — escalate to Sal/Rob.
- Cause #5: Run sunset flow. Set the criteria to suppress profiles with no opens/clicks in 180+ days. The list will shrink, but the remaining list will perform better.
- Cause #6: Re-segment. Make sure you're sending to engaged subscribers (opened in last 30/60/90 days) for high-volume campaigns. See email-marketing skill's "5 core segments" guidance.

**Prevention:** Run a weekly "deliverability check": bounce rate < 2%, complaint rate < 0.05%, open rate > 35%. If any cross the threshold, pause and investigate before continuing.

---

## Symptom 4: "My emails are going to spam"

Related to Symptom 3 but specifically about inbox placement. Same causes plus a few specific to spam-folder routing.

**Likely causes:**

1. Authentication failure (DKIM/SPF/DMARC) — same as Symptom 3 cause #1
2. Spam-trigger words in subject or body
3. Image-to-text ratio too high (mostly images, little text)
4. Low sender reputation (Gmail/Yahoo learn from past engagement)
5. Recipient marked previous emails as spam
6. Suspicious links (URL shorteners, redirects through unknown domains)
7. New sending domain without warm-up

**How to diagnose:**

1. Run a deliverability test (mail-tester.com or Klaviyo's Inbox Insights if available). It scores your email and tells you which factors hurt placement.
2. Look at the email's HTML source. Count the ratio of image bytes to text bytes. Heavy on images = cause #3.
3. Check whether the recipient ever marked a previous email as spam (Profile → Activity Feed → look for "Marked as spam"). That's cause #5.
4. Check links in the email. Any `bit.ly` or `t.co` or unknown shortener? That's cause #6.

**Fix per cause:**

- Causes #1, #2, #4: Same fixes as Symptom 3.
- Cause #3: Aim for 60% text / 40% image minimum. Add more text content. Use HTML for buttons instead of image buttons.
- Cause #5: That recipient is lost to your sender reputation. Suppress them.
- Cause #6: Use Klaviyo's link tracking instead of third-party shorteners. Klaviyo wraps links with its own tracking domain, which is reputable.
- Cause #7: New sending domains need 2-4 weeks of slow-volume warm-up. Start with 100 sends/day to most-engaged profiles, double weekly. Don't blast 1,000+ sends from a brand new domain.

**Prevention:** Test every campaign through mail-tester.com before sending. Aim for a score of 9/10 or higher.

---

## Symptom 5: "Two emails went to the same person on the same day"

**Likely causes:**

1. Smart Sending is off (most common)
2. Two flows are firing on the same trigger
3. A flow has a duplicate touch
4. The profile entered the flow twice (re-entry allowed)
5. The recipient has two profiles (two emails point to the same inbox)

**How to diagnose:**

1. Check Settings → Account → Smart Sending. Is it on? What's the threshold (default 16h email / 12h SMS)?
2. Check Audience → Profiles → search → Activity Feed. Look at the timestamps and source flow of each email. Same flow or different?
3. If same flow, open the flow editor. Is there an accidental duplicate touch?
4. If different flows, identify both. Are both supposed to fire on the same audience? See Section 6 of [05-go-live-checklist.md](./05-go-live-checklist.md) for conflict-checking.
5. If different profiles, look at both. Same email but different profile IDs?

**Fix per cause:**

- Cause #1: Turn Smart Sending on. Settings → Account → Smart Sending → Email: On (16h), SMS: On (12h).
- Cause #2: Add a flow exclusion filter to one or both flows ("Profile is not currently in [other flow]"). Or stagger the timing.
- Cause #3: Edit the flow. Delete the duplicate touch.
- Cause #4: Open the flow → Trigger box → Settings → "Allow re-entry?" — usually should be off. Turn it off.
- Cause #5: Merge the duplicate profiles (Klaviyo → select both profiles → Merge). Going forward, ensure your signup form prevents duplicates.

**Prevention:** Smart Sending always on. Review flow conflicts before activating any new flow. Use the conflict-check in the go-live checklist.

---

## Symptom 6: "SMS didn't deliver"

**Likely causes (ranked):**

1. Phone number invalid or formatted wrong (most common)
2. Country code missing or wrong
3. Recipient previously sent STOP (opt-out triggered)
4. Recipient's carrier is filtering Klaviyo
5. Klaviyo's SMS infrastructure has a temporary issue
6. The recipient's number is a landline, not mobile

**How to diagnose:**

1. Open the test profile. Look at the phone number field. Is it in E.164 format? `+15195551234` not `(519) 555-1234`.
2. Check the country code. For Canadian numbers, it should be `+1`. For US, `+1`. Other countries, the relevant code.
3. Check the profile's SMS subscription status. Settings → Profile → Subscriptions tab. If "Unsubscribed" or "Suppressed," the recipient previously opted out.
4. Check Klaviyo's status page (status.klaviyo.com) for any active SMS infrastructure issues.
5. Use a phone number lookup tool (e.g., truecnam.com) to check whether the number is mobile or landline.

**Fix per cause:**

- Cause #1: Reformat to E.164. Klaviyo will normalize on save, but the original entry should be E.164.
- Cause #2: Add the correct country code prefix.
- Cause #3: That profile is permanently SMS-suppressed. Cannot be re-subscribed without their explicit opt-in via a form.
- Cause #4: Open a Klaviyo support ticket. Provide the recipient's number and timestamp. Klaviyo will work with the carrier.
- Cause #5: Wait. Check status page. Most Klaviyo SMS issues resolve within 1-2 hours.
- Cause #6: Mark the profile as "no SMS — landline" in a custom property. Send only email going forward.

**Prevention:** Validate phone numbers at signup (Klaviyo Form has a built-in validator). Always include country code. Don't import phone numbers without verifying format.

---

## Symptom 7: "My click rate is 0% but open rate is high"

People are opening but not clicking. Either the email's content isn't compelling, the links are broken, or the links are filtered.

**Likely causes:**

1. Links are broken or go to a 404 page
2. Email is image-only with the link inside the image (some clients block image clicks)
3. Suspicious URL flagged by recipients' browsers
4. The CTA isn't clear or compelling
5. UTM parameters broke the URL

**How to diagnose:**

1. Open the email in your inbox. Click every link. Does it work?
2. Check the email's HTML. Is the CTA an image with a link, or a real HTML button?
3. Hover over the CTA link. What domain does it point to? Recognized (therunrec.com, klaviyo-tracked) or unknown?
4. Re-read the CTA copy. Is it clear what the recipient should click?
5. If the link includes UTM parameters (like `?utm_source=klaviyo&utm_medium=email`), test it without them. Does it work then?

**Fix per cause:**

- Cause #1: Edit the email, fix the link, send a Send Test, click to verify. For an in-flight flow, clone it, fix the link in the clone, swap the live flow with the clone. Then halt in-flight profiles in the broken version.
- Cause #2: Convert image-only CTAs to HTML buttons. They render better, work in image-blocked clients, and are clickable.
- Cause #3: Replace third-party shorteners with Klaviyo's link tracking. If the destination domain itself looks suspicious, fix the URL or use a real branded URL.
- Cause #4: Rewrite the CTA. Per email-marketing skill, use Kelsey's template: "Just a heads up that [X] is available — [click here] to grab it." Specific, low-pressure, contextual.
- Cause #5: Strip the UTM parameters and re-add one at a time until the URL works. The most common bug is duplicate `?` characters when both Klaviyo and the URL have query strings.

**Prevention:** Click every link in every email during testing (per the test protocol). Don't rely on visual review — actually click.

---

## Symptom 8: "A flow is firing but no one's getting emails"

The flow shows entries but no actual sends.

**Likely causes:**

1. Flow status is Draft, not Live
2. The first touch has a long delay (e.g., 24h) and you haven't waited
3. The first touch has a filter that's excluding everyone
4. Send time restrictions (quiet hours) are deferring all sends
5. The audience has Smart Sending blocking sends
6. Klaviyo email service is paused at the account level

**How to diagnose:**

1. Look at the flow's status indicator (top-left in the flow editor). Should be green "Live."
2. Open the first touch. Look at the delay before it. If it's more than a few minutes, that's why there's no immediate send.
3. Look at the first touch's filter (next to the delay). Is there a "Skip if X" rule?
4. Open the flow's analytics. How many entered? How many were sent? If entered = 100 but sent = 0, sends are being skipped.
5. Click on the per-touch analytics. Look for "Skipped" reasons.
6. Check Settings → Account. Is there an "Account paused" warning at the top?

**Fix per cause:**

- Cause #1: Change to Live. Manage Flow → Update flow status → Live.
- Cause #2: Wait the delay period, or change the delay if it's wrong. Don't change a delay on a Live flow without escalating — in-flight profiles will be affected.
- Cause #3: Open the filter. Re-evaluate. If it's excluding everyone, the filter logic is wrong.
- Cause #4: Wait until the next quiet-hours window opens (9 AM recipient timezone). Sends will fire in batch.
- Cause #5: Wait 16-24 hours for Smart Sending threshold to reset.
- Cause #6: Check billing. Klaviyo pauses accounts for unpaid invoices. Escalate to Sal.

**Prevention:** Always Send a test of the first touch immediately after activation to verify sends are firing.

---

## Symptom 9: "How do I see who's currently in a flow?"

Not a problem — a common question. Here's how.

**Where to find:**

1. Open the flow in the editor
2. Click the **Analytics** tab (top-right area)
3. Look for the "People in flow" or "Active recipients" widget
4. Click it for a list of every profile currently somewhere in the flow

You can also see per-touch:

1. In the flow editor, click any touch
2. Look at the touch's "Recipients" or "Sent to" panel
3. See exactly who received that specific email/SMS and when

For a single profile's flow status:

1. Open the profile (Audience → Profiles → search by email)
2. Look at the **Activity Feed**
3. Search for the flow name to see entry/exit and per-touch records

---

## Symptom 10: "How do I remove someone from a flow without unsubscribing them?"

Unsubscribe is permanent. Removing from a flow is a single-flow exit.

**How to do it:**

1. Open the profile (Audience → Profiles → search by email)
2. Click the **Activity Feed** tab
3. Find the entry "Entered flow [Flow Name]"
4. To the right of that entry, click **Actions** → **Exit profile from this flow**
5. Confirm. Profile exits immediately. They will not receive any further touches in this flow.

The profile remains subscribed to email/SMS marketing in general. They can still enter other flows.

For bulk operations:

1. Open the flow → Analytics → People in flow
2. Filter to the subset you want to exit
3. Bulk action → Exit profiles from flow

---

## Symptom 11: "How do I delete a flow?"

Deleting a flow is destructive. Klaviyo prevents you from deleting a Live flow.

**The correct sequence:**

1. **Pause first.** Manage Flow → Update flow status → Draft. New profiles stop entering.
2. **Wait 7+ days.** Existing profiles in flight finish their journey. (For long flows, this could be weeks — adjust accordingly.)
3. **Confirm zero people in flow.** Open Analytics → People in flow → should be 0.
4. **Archive (not delete).** Manage Flow → Archive flow. This hides the flow from the active list but preserves all historical data (sends, opens, clicks, attribution).

**Never click Delete.** Klaviyo's archive is reversible. Delete is not. Archive serves the same purpose without the risk.

---

## Symptom 12: "Klaviyo shows my flow has errors"

Klaviyo validates flows when you save and warns about specific issues. Common error messages and what to do.

| Error message | Meaning | Fix |
|--------------|---------|-----|
| "Trigger requires X but no events match" | The metric you selected has zero events. Trigger won't fire. | Verify the metric is being tracked. Check Metrics → search for your metric → see recent events. If empty, the integration isn't firing. |
| "Filter references property X that doesn't exist" | A filter checks a profile property that isn't created in Klaviyo. | Settings → Profile Properties → create the property. Or remove the filter. |
| "Conditional split has unreachable branch" | One branch of a split can never be hit because of upstream filters. | Re-check the conditional logic. Usually means a filter is too strict. |
| "Email template missing required content" | Email body has empty fields, broken merge tag, or no unsubscribe link | Open the email, fix the missing element. Klaviyo highlights what's missing. |
| "SMS exceeds 160 character limit" | Your SMS will split into multiple segments | Shorten the message, or accept the multi-segment cost (you'll be charged per segment) |
| "Sender profile not configured" | The from-name/from-email on the email isn't set | Edit the email → Settings → Sender Profile → choose one |
| "Audience size exceeds X" | Triggering this flow would hit too many people too fast | Either narrow the audience or schedule via a campaign instead of a flow |

**Always read the error message in full.** Klaviyo tells you exactly what to fix.

---

## Symptom 13: "How do I roll back a bad change?"

Klaviyo has version history for flows and emails. Use it.

**For flows:**

1. Open the flow
2. Click **Manage Flow** → **Version History**
3. See a list of every save, with timestamp and author
4. Click any version to see what it looked like at that time
5. Click **Restore** to revert the flow to that version
6. Confirm. The flow snaps back to that state.

**For emails (within a flow):**

1. Open the email touch
2. Click the email name to enter the editor
3. Top-right → **History** or **Version History**
4. Same workflow — choose a version, click Restore

**For campaigns:**

Campaigns have version history too. Same pattern: open campaign → Version History → Restore.

**Caveats:**

- Restoring a flow does not undo sends that already happened. If a bad version sent emails, those emails are out.
- Restoring while profiles are in-flight in a Live flow can have unexpected effects. Pause the flow first, restore, then re-activate.
- Version history typically goes back 30-90 days depending on Klaviyo plan. Don't rely on it for older changes.

**Prevention:** Always note the date you make a significant change in the flow's Notes panel. If you have to roll back, you'll know which version to choose.

---

## Symptom 14: Other situations not yet documented

If you're seeing something that isn't here, here's the diagnostic order:

1. **Open the relevant profile's Activity Feed.** This tells you what Klaviyo did or didn't do.
2. **Check the flow's analytics.** Skipped reasons, send counts, error counts.
3. **Check Settings → Account → Status messages.** Any account-level alerts?
4. **Check Klaviyo's status page** (status.klaviyo.com) for infrastructure issues.
5. **Search the Klaviyo Help Center** (help.klaviyo.com) for the symptom.
6. **Open a Klaviyo support ticket** if you can't resolve in 30 minutes.
7. **Escalate to Sal** with screenshots, the profile email (if any), and the timestamp.

---

## Final reminder

Most "weird" Klaviyo behavior comes from a small set of root causes:

- Trigger doesn't match the actual event
- Filter excludes more than intended
- Smart Sending is interacting with another flow
- Authentication is broken (DKIM/SPF/DMARC)
- Sender reputation is damaged
- Profile is suppressed or has missing data

Walk through these in order before assuming Klaviyo is broken. Klaviyo is rarely the problem. Configuration usually is.

When in doubt, send a test, check the activity feed, and ask Sal.

Back to: [README.md](./README.md)
