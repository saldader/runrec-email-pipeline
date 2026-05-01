# Voice & Mechanic Doctrine — Quick Reference

**Purpose:** When you're building a flow and the per-flow guide says "use Sal & Izzy – RunRec Co-Owners signature," check this card to confirm WHEN that signature is right vs wrong. If a per-flow guide and this card disagree, **this card wins**.

---

## Voice Doctrine — Who Signs What

There are four signatures in active rotation across the 13 flows. Each has a specific use.

| Signature | When to use it | Examples |
|-----------|---------------|----------|
| **Sal & Izzy – RunRec Co-Owners** | Founder warmth moments. Family/community/long-relationship touches. The reader should feel like the people who built RunRec are personally talking to them. | Welcome Sec 1 T01 (What you signed up for), T02 (Origin story), T06 (Behind the scenes); Anniversary Sec 5 T01 (Memory trigger); Lifecycle Sec 11 T07 (Door's always open) |
| **Sal – RunRec Co-Owner** | Personal closes and high-touch consults. The reader should feel like a real person is reaching out one-to-one — not the brand, not the team. | Anniversary Sec 5 T02/T03/T05 (early lock, personal nudge, decision-force close); Lead Nurture Sec 8 T03 (Conversation opener SMS); Corporate Sec 9 T01/T03/T05 (Personal-feel reply, Case study, Quick question re-open); Lifecycle Sec 11 T08 (Come back: 50% off) |
| **{{location_manager}}** | Multi-location B2B. The signature merge-tag resolves to the local manager's name (e.g., "Aren" for Burlington). Body uses {{location_manager}} too; From: header uses {{location_manager}}@therunrec.com. | VIP Sec 7 T01 (Status conferral); Corporate Sec 9 — ALL touches (T01-T07 from-header + sender-note panel; T07 body + signature) |
| **The RunRec Team** | Everything else. Operational, automated, informational, list-hygiene. Default voice when none of the above apply. | All other touches across all flows. Default fallback. |

### When in doubt: use **The RunRec Team**

Default to the team voice. The other three are exceptions for specific moments.

---

## Mechanic Doctrine — How Customers Redeem Offers

**Codes are deprecated.** No new flow uses a discount code. Do not invent one.

| Old mechanic (DO NOT USE) | New mechanic (USE THIS) |
|---------------------------|--------------------------|
| `Code FIRST25` (Lead Nurture first session) | "Reply with your preferred day and time and we'll redeem the offer for you" |
| `Code BDAY-{year}` (Birthday gift hour) | "Screenshot this email/SMS and reply — we'll book your free hour" |
| `Code COMEBACK / RETURN25` (Win-Back) | "Reply with your ideal day and time and we'll reserve it for you" |
| `Code 50OFF / WELCOME` (Lifecycle) | "Reply YES and we'll connect you with the membership team" / "First month is on us — just say the word" |
| Any URL like `runrec.co/c/CODE` (auto-redeem links) | Reply or screenshot — handled by inbox triage |

### Why we changed

1. **Inbox triage builds relationships.** Every reply is a conversation. Every code redemption is a click and forget.
2. **Codes get shared.** A reply mechanic is one-to-one. A code on Twitter is to everyone.
3. **Conversion rates are higher.** People feel committed once they've replied. The friction is good friction.

### What this means operationally

Someone has to monitor `contact@therunrec.com` and the SMS reply inbox. Target SLA: **4 hours during business hours.** If no one is monitoring, do NOT activate Flows 04 (T02), 06 (T02/T04/T05), 08 (T06/T07), 09 (T01/T03/T04/T06), 10 (T03/T04), 11 (T05/T06), or 12 (T01).

---

## Sender Address Map

Per-flow sender addresses. From: header configuration in Klaviyo.

| Flow | From: name | From: email | Notes |
|------|-----------|-------------|-------|
| 01 Welcome | The RunRec | contact@therunrec.com | Consumer touchpoint |
| 02 Booking Confirm | The RunRec | contact@therunrec.com | Transactional |
| 03 Post-Session | The RunRec | contact@therunrec.com | Consumer transactional + nudge |
| 04 Customer Birthday | The RunRec | contact@therunrec.com | Consumer celebratory |
| 05 Anniversary | The RunRec | contact@therunrec.com | Consumer celebratory |
| 06 Win-Back | The RunRec | contact@therunrec.com | Consumer re-engagement |
| 07 VIP T01 | {{location_manager}} | {{location_manager}}@therunrec.com | Personal status award |
| 07 VIP T02-T05 | The RunRec | contact@therunrec.com | Other VIP touches |
| 08 Lead Nurture | The RunRec | contact@therunrec.com | EXCEPT Touch 03 SMS sender label = "Sal – RunRec Co-Owner" |
| 09 Corporate ALL | {{location_manager}} | {{location_manager}}@therunrec.com | B2B requires personal sender per location |
| 10 Membership Conversion | The RunRec | contact@therunrec.com | Consumer conversion |
| 11 Lifecycle | The RunRec | contact@therunrec.com | Member touchpoints |
| 12 Abandoned Booking | The RunRec | contact@therunrec.com | Consumer recovery |
| 13 Sunset | The RunRec | contact@therunrec.com | List hygiene |

### Multi-location merge tag handling

`{{location_manager}}` and `{{location_manager}}@therunrec.com` are **profile properties**, not literal text. Klaviyo will substitute them per-recipient based on their `location_manager` and `location_manager_email` profile properties.

For the Waterloo HQ rollout (only location currently active), set the default values to:
- `location_manager` = "Sal"
- `location_manager_email` = "sal@therunrec.com"
- `location_phone` = "519-800-9894"

Future locations populate per their own manager.

---

## Default sign-off conventions

When the per-flow guide says "signed The RunRec Team", use these exact strings:

| Channel | Format |
|---------|--------|
| Email body | `— The RunRec Team` (em-dash + space + "The RunRec Team") |
| SMS body | `-The RunRec Team` (hyphen + "The RunRec Team") |
| SMS sender label | `RunRec` (just the brand name, no "Team" — sender labels are tight) |

For founder voice:

| Channel | Format |
|---------|--------|
| Email body | `— Sal & Izzy – RunRec Co-Owners` |
| SMS body | (rare in founder voice — usually email-only) |

For personal closer voice:

| Channel | Format |
|---------|--------|
| Email body | `— Sal – RunRec Co-Owner` |
| SMS body | `-Sal – RunRec Co-Owner` |
| SMS sender label | `Sal – RunRec Co-Owner` |

For location manager voice:

| Channel | Format |
|---------|--------|
| Email body | `— {{location_manager}}` (just the merge tag, no role descriptor — the From: address makes it clear who they are) |
| SMS body | (Sec 9 T02 only — `-The RunRec Team` per spec) |

---

## Subject line voice rules

- **Use "we" not "I"** — applies to every flow except where Sal personally signs (then "I" is fine in body, not in subject)
- **No exclamation marks** in subjects
- **No "$" or "FREE"** — spam triggers
- **Under 50 characters** — preview text continues the hook

Example: ✅ "About that first session." (curiosity-driven, team voice)
Example: ❌ "First session 25% OFF — only this week!" (price-driven, multiple spam triggers)

---

## Quick decision flowchart

You're writing/checking a touch's signature. Walk through this:

1. Is this a founder warmth moment (welcome, anniversary, member family)?
   → Yes: **Sal & Izzy – RunRec Co-Owners**
2. Is this a personal one-to-one nudge (close, consult, re-engage)?
   → Yes: **Sal – RunRec Co-Owner**
3. Is this a corporate B2B touch where the local manager owns the relationship?
   → Yes: **{{location_manager}}**
4. Anything else?
   → **The RunRec Team**

---

## Sources

- Master architecture: `~/vault/runrec/marketing/klaviyo-master-buildsheet-v2.md` Section 00.5 (Voice Doctrine) + Section 00.6 (Mechanic Doctrine)
- Operator's adjustments doc: `~/vault/runrec/marketing/adjustments-flows-2026-05-01.md`
- Audit verification: `~/vault/runrec/marketing/audit-2026-05-01.md`

If a per-flow guide contradicts this card, this card is the canonical reference.
