# 06 — Compliance and Precautions

**Audience:** The admin who is sending RunRec emails and SMS
**Purpose:** Plain-English explanation of legal requirements + how Klaviyo's settings satisfy them
**When to use this:** Read once early in the build process. Reference whenever a specific compliance question comes up.

> **This document is best-practice guidance, not legal advice.** Compliance laws are interpreted by courts and regulators. For specific legal questions — especially before launching a new program type, before sending to a new region, or after a complaint — consult a Canadian or US attorney with marketing-law experience. The framework below reflects how Klaviyo's settings map to common requirements; it is not a substitute for a legal review of your specific situation.

---

## Why compliance matters for RunRec

RunRec is:

1. **A Canadian business** — primarily subject to **CASL** (Canada's Anti-Spam Legislation)
2. **Operating in Ontario** — subject to provincial privacy law (PIPEDA equivalent)
3. **Sending some email to US contacts** — subject to **CAN-SPAM** for those recipients
4. **Sending SMS** — subject to Canadian SMS rules under CASL plus US **TCPA / CTIA** guidelines for US numbers
5. **Potentially handling EU contacts** — even a few EU-resident customers triggers **GDPR** obligations

The penalties are not theoretical:

| Law | Maximum penalty |
|-----|----------------|
| CASL | Up to $10 million CAD per violation (corporations); $1 million per violation (individuals) |
| CAN-SPAM | Up to $51,744 USD per individual email (yes, per email — not per send batch) |
| TCPA (US SMS) | $500 to $1,500 per text message in violation |
| GDPR | Up to €20 million or 4% of global revenue, whichever is greater |

These are enforced. Regulators do bring cases. The cost of compliance is small; the cost of a violation is business-ending.

The good news: Klaviyo handles most of the heavy lifting if it's configured correctly. Below is what's required, and how Klaviyo's defaults satisfy it.

---

## Section 1 — CASL (Canada's Anti-Spam Legislation)

Authority: **Canadian Radio-television and Telecommunications Commission (CRTC)** — https://crtc.gc.ca/eng/internet/anti.htm

### What CASL requires

CASL applies to "Commercial Electronic Messages" (CEMs) — any email, SMS, or social-media DM with a commercial purpose sent to or from Canada. To send a CEM legally, you need:

1. **Consent** from the recipient — either express or implied
2. **Identification** of the sender (business name and physical address)
3. **An unsubscribe mechanism** that is functional and processed within 10 business days

### What "consent" means under CASL

| Consent type | What it is | How long it lasts |
|--------------|-----------|-------------------|
| **Express consent** | Recipient actively opted in (checked a box, submitted a form, replied YES) | Forever, until they unsubscribe |
| **Implied consent — existing business relationship** | Recipient bought from you in the last 2 years, or inquired in the last 6 months | Time-limited (24 months for purchases, 6 months for inquiries) |
| **Implied consent — published address** | Recipient has published their email publicly without a "no spam" notice, AND your message is relevant to their role/business | While address remains published |

For RunRec specifically:

- **Newsletter list (1,435 profiles):** These signed up via the website footer or a Klaviyo form. Express consent.
- **Skedda Email List (767 profiles):** These booked a court via Skedda. Implied consent (existing customer) for up to 24 months from their last booking. After that, they need express consent or you have to stop emailing.
- **Tri-City Member Emails (63 profiles):** Active members with ongoing business relationship. Express consent assumed (verify in member onboarding paperwork).
- **Birthday Customers (19 profiles):** Implied consent for 24 months from last party.
- **Previous Campers (109 profiles):** Implied consent — but if their last engagement was over 24 months ago, you need to either (a) get express consent before sending more, or (b) stop sending.

**The 24-month rule is the trap.** Customers from 2022 or earlier may now be outside the implied-consent window. Klaviyo doesn't enforce this automatically. If you're unsure, run a re-consent campaign before sending to long-dormant customers, or migrate the flow to "express consent only."

### What "identification" means under CASL

Every commercial email and SMS must clearly identify:

- The sender's business name (e.g., "RunRec")
- A current physical mailing address (PO box is acceptable; the Northfield Dr. E address satisfies this)
- Contact information for the sender (email or phone)

**How Klaviyo handles this:** Klaviyo's email footer template auto-populates the business name and physical address from your account settings. Verify in **Settings → Account → Organization Information** that:

- Organization name is "RunRec" (or legal entity name if different)
- Mailing address is current (the audit confirms 283 Northfield Dr. E Unit 13, Waterloo, ON N2J 4G8)
- Phone number is set

Every email template should reference these via `{{ organization.name }}` and `{{ organization.full_address }}` merge tags.

### What "unsubscribe mechanism" means under CASL

Every CEM must include:

- A clear unsubscribe link or instructions
- The mechanism must work (not a 404)
- The unsubscribe must be honored within 10 business days
- The unsubscribe must be free (no fee, no required login if not previously required)

**How Klaviyo handles this:** Klaviyo automatically adds an unsubscribe link to every email footer. Clicking it takes the recipient to a hosted unsubscribe page. The profile is marked as unsubscribed within seconds. Klaviyo will then refuse to send future marketing emails to that profile across all flows and campaigns.

**You do not have to build the unsubscribe page.** It's automatic. Just don't disable it. Verify in **Settings → Account → Email Settings** that "Include Klaviyo unsubscribe link in all emails" is on.

For SMS: Klaviyo automatically processes "STOP" as an unsubscribe and sends a confirmation. Same as email — automatic, just don't disable. See Section 3 for additional SMS-specific requirements.

### CASL-specific best practices for RunRec

- **Document consent.** Klaviyo records `$consent`, `$consent_timestamp`, `$consent_method`, `$consent_form_id` for every signup. Don't disable these. They are your proof of compliance if a complaint is investigated.
- **Honor unsubscribe immediately.** Klaviyo does this automatically — don't override it.
- **Never re-add an unsubscribed profile to a list manually** without re-getting express consent through a form.
- **Run a re-consent campaign annually** for profiles approaching the 24-month implied-consent window.
- **Don't email unverified imports.** If a list was imported from a source you can't document the consent for (e.g., an old spreadsheet of unknown origin), do not send to it. You may be liable for sending to non-consenting addresses.

---

## Section 2 — CAN-SPAM (United States)

Authority: **US Federal Trade Commission (FTC)** — https://www.ftc.gov/business-guidance/resources/can-spam-act-compliance-guide-business

CAN-SPAM applies whenever you send commercial email *to a US recipient*. Even a small US contact list triggers compliance for those addresses.

### What CAN-SPAM requires

CAN-SPAM is generally **less strict than CASL** on consent (it allows opt-out instead of opt-in), but stricter on a few content rules:

1. **No deceptive subject lines.** The subject must accurately reflect the email's content.
2. **No deceptive header information.** From, To, and Reply-To addresses must accurately identify the sender.
3. **Identification as advertisement.** If the email is purely commercial, it must be identifiable as such (CASL doesn't require this; CAN-SPAM does).
4. **Working unsubscribe mechanism.** Must process within 10 business days.
5. **Physical address.** Same requirement as CASL.
6. **Honor opt-outs across the business.** Once a recipient unsubscribes, no marketing emails from any RunRec property should reach them.

### How Klaviyo handles CAN-SPAM

The same Klaviyo configuration that satisfies CASL also satisfies most of CAN-SPAM. The exception is the "identification as advertisement" rule for purely commercial messages. RunRec's brand voice already softens promotional emails into editorial-style content (e.g., "What you just signed up for," not "BUY NOW 50% OFF"), which mostly avoids the strict-promotional category.

If you ever send a hard-sell promotional email to US recipients, add wording like "This is a promotional email from RunRec" near the top. Subtle and one-time, but covers the requirement.

---

## Section 3 — SMS / TCPA / CTIA / CASL SMS rules

SMS has stricter consent rules than email almost everywhere.

### Canada — CASL SMS

CASL applies to SMS the same way it applies to email. Express or implied consent required. Identification and unsubscribe required.

**Differences from email:**

- The unsubscribe mechanism is **STOP** (or HALT, QUIT, UNSUBSCRIBE — Klaviyo recognizes all). It must work without requiring a paid call or web visit.
- The first SMS in any flow should explicitly tell the recipient how to opt out: "Reply STOP to opt out."
- Identification (sender name) must be in the message itself (since SMS has no headers): include "RunRec" or similar in the message body or sender ID.

### United States — TCPA + CTIA

US SMS is governed by:

- **TCPA** (Telephone Consumer Protection Act) — federal law requiring **express written consent** for marketing SMS to mobile numbers
- **CTIA** (Cellular Telecommunications Industry Association) — industry guidelines that wireless carriers enforce; non-compliance gets you blocked at the carrier level

TCPA's "express written consent" requirement is the strictest in this document. To send marketing SMS to a US mobile number legally, you need:

- **Recipient's signature** (electronic checkbox is fine) on a disclosure that:
  - Identifies the sender (RunRec)
  - States the purpose (marketing messages, transactional reminders, etc.)
  - Includes the message frequency ("up to N messages per month")
  - Includes "Message and data rates may apply"
  - Includes opt-out instructions ("Reply STOP to opt out, HELP for help")

Klaviyo's standard SMS opt-in form covers this. **Do not modify Klaviyo's SMS consent flow without legal review.**

### CTIA quiet hours (recommended even where not legally required)

CTIA recommends not sending marketing SMS between **9 PM and 9 AM in the recipient's local timezone**. Klaviyo's Quiet Hours setting enforces this automatically. Verify it's on (Settings → Account → Quiet Hours).

### How Klaviyo handles SMS compliance

| Requirement | How Klaviyo handles it | Your job |
|------------|----------------------|---------|
| STOP keyword | Auto-processed; sends confirmation; profile marked unsubscribed | Don't disable |
| HELP keyword | Auto-replies with contact info | Verify the auto-reply text in Settings → SMS |
| First-message opt-out language | Klaviyo prompts you to include it; doesn't enforce | Add "Reply STOP to opt out" to the first SMS in every flow |
| Quiet hours | Auto-enforced if enabled | Enable in Settings → Account → Quiet Hours |
| Consent record | Auto-recorded with timestamp + form ID | Verify in profile properties |
| Carrier blocking | Klaviyo monitors and reports if a carrier blocks your messages | Watch for alerts in dashboard |

### SMS rules specific to RunRec's flows

The audit notes that current SMS messages don't include "Reply STOP" copy in most flows. Per CASL best practice, **add "Reply STOP to opt out" to the first SMS in each flow** even if Klaviyo's STOP keyword works automatically. It reduces complaint risk and makes compliance visible.

The 12 flows that need this audit:

- Welcome (Touch 04)
- Booking Confirm (Touch 02)
- Post-Session (Touch 01)
- Birthday (Touch 02)
- Anniversary (Touch 03)
- Win-Back (Touch 03)
- VIP (Touch 04)
- Lead Nurture (Touch 03)
- Corporate (Touch 02)
- Membership Conversion (Touch 04)
- Membership Lifecycle (Touch 06)
- Abandoned Booking (Touch 03)

Each first-SMS should be reviewed and updated before launch.

---

## Section 4 — GDPR (if any EU contacts)

Authority: **European Data Protection Board** — https://edpb.europa.eu/

GDPR applies if RunRec collects, processes, or stores personal data of any individual physically located in the EU at the time of data collection. Even a single EU resident triggers obligations.

For RunRec, this is **probably low-risk** (Tri-Cities, Ontario customer base), but worth knowing:

### Core GDPR rights

| Right | What it means | How to handle |
|-------|--------------|---------------|
| Right to access | EU residents can request a copy of all data you have on them | Use Klaviyo's profile export feature; respond within 30 days |
| Right to delete (right to be forgotten) | EU residents can request deletion | Use Klaviyo's profile delete; complete within 30 days |
| Right to rectification | EU residents can request corrections | Edit the profile in Klaviyo |
| Right to data portability | EU residents can request their data in a machine-readable format | Klaviyo's export gives JSON/CSV |
| Right to restrict processing | EU residents can ask you to stop using their data without deleting it | Suppress the profile in Klaviyo (keeps data, prevents sends) |

### Lawful basis for processing

Under GDPR, you need a "lawful basis" to process personal data. For marketing, this is almost always **consent** (Article 6.1.a). Klaviyo's consent records satisfy this if the signup form was GDPR-compliant.

### What to do if an EU resident emails you with a GDPR request

1. Acknowledge receipt within 5 business days
2. Verify the requester's identity (email match, ideally a second factor)
3. Complete the request within 30 days
4. Document everything in case of regulator audit

**If unsure, escalate to Sal.** GDPR requests are not a normal support ticket.

---

## Section 5 — Suppression list management

A "suppression list" is the master list of profiles who must not receive marketing. Klaviyo maintains this automatically.

### What goes on the suppression list

- **Unsubscribed profiles** — opted out of marketing
- **Bounced addresses** — hard bounces (invalid email) and repeated soft bounces
- **Spam complaints** — recipients who marked an email as spam in their inbox
- **Manual suppressions** — profiles you suppress directly (e.g., legal request)

### What Klaviyo does automatically

- Suppresses unsubscribers immediately
- Suppresses hard bounces immediately
- Suppresses soft bounces after a configurable threshold (default: 7 — the email-marketing skill recommends lowering this to 2-3 for better list hygiene)
- Suppresses spam complaints immediately
- Refuses to send to suppressed profiles, even if added to lists/segments by mistake

### What you should never do

- Never manually un-suppress a profile (Settings → Suppressions → Remove). Once a profile is suppressed, it stays suppressed forever unless they re-opt-in via a form. The only way to legitimately re-engage a suppressed profile is for them to actively re-subscribe.
- Never import a list that includes previously-suppressed addresses. Klaviyo will block them, but the import attempt itself is a flag.
- Never share suppression lists between businesses. Each company's suppression list is bound to that company's consent records.

---

## Section 6 — Profile data hygiene

Beyond compliance laws, there are best practices that protect RunRec's brand and sender reputation.

### Don't

- **Don't import scraped lists.** Email addresses bought, scraped, or exchanged with another business are not consenting to RunRec. Sending to them violates CASL and torches sender reputation.
- **Don't send to addresses you can't document the source of.** If you can't say "this person signed up via X form on Y date," don't send marketing to them.
- **Don't use one customer's data to message another.** Example: don't send a "your friend Hassan recommends RunRec" email using Hassan's name without Hassan's permission.
- **Don't reactivate a customer who unsubscribed by adding them back manually.** They have to actively re-subscribe through a form.
- **Don't send promotional content to transactional-only opt-ins.** A customer who opted in for booking confirmations did not opt in for newsletters. Keep flows separate.

### Do

- **Do honor opt-out forever.** Once suppressed, always suppressed (unless they actively re-subscribe).
- **Do document consent.** Klaviyo records this automatically. Don't disable it.
- **Do clean the list.** Run a sunset/hygiene flow (Flow 13) to remove unengaged profiles. Sending to dead addresses tanks deliverability.
- **Do segment carefully.** "Newsletter signups" and "paying customers" should be different segments with different content rules.
- **Do separate marketing from transactional.** Booking confirmations, password resets, payment receipts are transactional and don't require marketing consent. But never bundle marketing content into a transactional email — it converts the email to marketing and you need full consent.

---

## Section 7 — Klaviyo audit log

For compliance audits and internal reviews, Klaviyo logs every meaningful action.

### Where to find it

**Settings → Account → Audit Log** — shows:

- Who logged in and when
- What flows were created, edited, activated, deactivated
- What profiles were imported, deleted, suppressed
- What campaigns were sent

**Per-profile activity** — Open any profile → Activity Feed — shows:

- Every email/SMS sent to them
- Every event they triggered
- Every flow they entered/exited
- Every list/segment they were added to or removed from
- Every consent change (subscribed, unsubscribed, opted in, opted out)

### How to use it for compliance audits

If a regulator (CRTC for CASL, FTC for CAN-SPAM) sends a complaint, you need to prove:

1. The complainant consented (when, how, what form)
2. The unsubscribe was processed (when, by what mechanism)
3. No marketing was sent after unsubscribe

The audit log + per-profile activity feed gives you all three. **Save screenshots immediately when a complaint comes in** — don't rely on the data being there later.

---

## Section 8 — Privacy policy

You are required by both CASL and most provincial privacy laws to publish a privacy policy that explains:

- What data RunRec collects (email, name, phone, booking history)
- How RunRec uses it (marketing, service delivery, analytics)
- Who RunRec shares it with (Klaviyo, Stripe, Skedda)
- How users can access, correct, or delete their data
- Contact info for privacy questions

### Recommendation

- Link to the privacy policy from the Klaviyo email footer ("Privacy Policy" next to "Unsubscribe")
- Link to it from every signup form
- Update it whenever you add a new data integration (e.g., when Skedda starts firing booking events into Klaviyo, the policy should mention Skedda)

If RunRec doesn't currently have a privacy policy, **escalate to Sal** — this is a pre-launch requirement, not a nice-to-have. A simple template-based privacy policy is acceptable for now; a tailored one can come later.

---

## Compliance quick-reference checklist

Run this every quarter, or before launching any new flow:

- [ ] Sender domain authentication (DKIM, SPF, DMARC) all green
- [ ] Organization name + physical address in Klaviyo settings (Settings → Account)
- [ ] Unsubscribe link in every email footer (auto via Klaviyo, verify it's not disabled)
- [ ] "Reply STOP" language in first SMS of every flow
- [ ] Quiet hours enforced (9 PM to 9 AM recipient timezone)
- [ ] Smart Sending enabled
- [ ] Privacy policy live and linked from email footer + signup forms
- [ ] No imported lists with undocumented consent
- [ ] No flows targeting profiles whose implied-consent window expired
- [ ] Suppression list reviewed (Settings → Suppressions) — not manually unsuppressing anyone
- [ ] Klaviyo audit log accessible (don't disable)

---

## When to escalate

Compliance gets murky in these cases. Always escalate to Sal (and from Sal to legal counsel if needed):

- A regulator (CRTC, FTC, EU data authority) contacts the business
- A customer formally complains about email/SMS frequency or accuracy
- A customer claims they didn't consent to receive what they got
- You discover a list import or signup form that may not have collected proper consent
- You're considering sending to a new audience (new region, new demographic, new acquisition source)
- You're considering a new SMS use case (e.g., transactional reminders for non-customers)
- You're considering buying or partnering with another business and inheriting their list
- A team member asks you to disable unsubscribe handling, suppression, or consent tracking

When in doubt, **don't send**. The cost of waiting a day to get a legal opinion is zero. The cost of sending a non-compliant message is potentially business-ending.

---

## Final note

The shape of these rules is similar across jurisdictions: get consent, identify yourself, honor opt-out, keep records. Klaviyo's defaults are designed around these requirements — keep the defaults on and you're 90% compliant.

The remaining 10% is your job: documenting consent sources, separating transactional from marketing, watching for the 24-month implied-consent expiry, adding "Reply STOP" to first SMS sends, and escalating murky cases to Sal and to legal counsel.

Compliance is not a one-time setup. It's a quarterly review.

Next: [07-troubleshooting.md](./07-troubleshooting.md) when something looks wrong.
