# Current Klaviyo State — Inventory as of 2026-05-01

**Reviewer:** CMO (read-only inventory, trust = SHADOW for write tools)
**Source:** Klaviyo MCP API (live read, 2026-05-01)
**Cross-reference:** `~/vault/runrec/marketing/klaviyo-master-buildsheet-v2.md` (the 13-flow plan)

---

## Account-level basics

- **Klaviyo organization name / ID** — Not exposed via the MCP tools available; need operator to confirm from Klaviyo UI Settings → Account.
- **Sender domain authentication** — Cannot be verified through MCP. Operator must confirm DKIM/SPF/DMARC + custom sending domain status in Settings → Domains.
- **Total profiles (with email-marketing consent)** — **2,989** (segment "Everyone" with email consent filter, ID `SrrkxP`). Note: the operator profile mentioned ~3,616 total profiles — the delta is unconsented/suppressed/SMS-only profiles.
- **Active integrations** (confirmed firing events):
  - **Stripe** (Payments) — `Successfully Paid`, `Failed Payment`, `Refunded Payment`, `Issued Invoice`. This is the de-facto e-commerce signal source.
  - **Shopify** (eCommerce) — `Placed Order` metric exists (ID `Y8rW4s`) but has zero attached properties; appears registered but not actively writing events. Needs verification.
  - **API** (custom) — `Active on Site`, `Viewed Product`, `Form submitted/completed by profile`. Likely Klaviyo JS snippet on website + form embeds.
  - **Klaviyo internal** — All system events (Opens, Clicks, Bounces, Unsubs, SMS lifecycle).
- **NOT integrated** (per inventory): Skedda (the booking system) does NOT appear to fire native events into Klaviyo. There is a "Skedda Email List" (767 profiles) but no `Booking Created` / `Booking Started` / `Booking Completed` metric. **This is the single biggest gap for the planned flows.**

---

## Lists (12 total)

| Name | ID | Count | Opt-in | Purpose / used by |
|------|----|-------|--------|-------------------|
| Newsletter | SDFmeU | **1,435** | single | Primary marketing list. Triggers New Subscribers segment + Welcome Flow `RZsSVX`. The de-facto "main list." |
| Skedda Email List | WyA26L | **767** | double | Imported from booking system (Skedda). Static — no flow attached. Largest pool of actual customers. |
| Previous Campers | RVpxwS | 109 | double | Camp/program alumni. No active flow. |
| Principle Emails | SQQBdj | 109 | double | Principal/admin contacts (likely school principals for Jr. NBA outreach). No active flow. |
| Tri-City Member Emails | XbJSh7 | 63 | double | Tri-City Members (the actual member roster). No active membership flow attached. **Plan-critical.** |
| Camp Waitlist (Summer 2025) | ULfg93 | 58 | double | Stale — Summer 2025 already passed. Candidate to archive. |
| Jr. NBA Emails | StczEH | 48 | double | Jr. NBA program contacts. Static. |
| SMS Recipients | UjrHmK | 38 | double | SMS-only consenters. Triggers SMS welcome sequence `QZXWNA`. |
| PlayForever Signups | TS6RMf | 23 | double | New (Nov 2025) — PlayForever program. No flow attached. |
| Birthday Customers | T8HDuV | 19 | double | Birthday party customers. **No automation attached** (plan calls for Flow 04 Customer Birthday + Flow 05 Anniversary on this list). |
| PlayForever League Emails | XHBsJD | 12 | double | New (Nov 2025) — PlayForever league. No flow attached. |
| Preview Test (Our Email) | Y2WZLm | 1 | double | Internal preview list. Keep, ignore. |

**Notes on lists:**
- The **Newsletter** list is single-opt-in — the only one. CASL exposure: low risk (Canada permits express + implied consent, and existing customers qualify), but the next agent should consider migrating to double opt-in for new signups and tagging legacy contacts as legacy-implied-consent.
- The **Skedda Email List** is a 767-profile pool of actual paying customers with no flow attached. This is the best target for plan implementation (it maps to "Court Bookers" in the plan).
- **Birthday Customers, Tri-City Members, Previous Campers, Jr. NBA, PlayForever (x2), Principle Emails** = 7 segment-like lists with zero automation. The plan's flows 04, 05, 09, 10, 11 should be wired to these once flows are built.

---

## Segments (2 total)

| Name | ID | Count | Definition | Status |
|------|----|-------|-----------|--------|
| Everyone | SrrkxP | 2,989 | Has email marketing consent (any subscription state) | Active, not starred. The de-facto "marketable list." |
| New Subscribers | TP6c8L | 14 | Newsletter list (SDFmeU) member in last 14 days AND has email consent | Active, **starred**. Maps to plan's "New Subscribers" segment. |

**Segment posture:** The account is essentially **un-segmented**. The CMO skill's 5 core engagement segments (30/60/90-day Engaged, Unengaged 90+, New Subscribers) are missing four of five. The plan's 8+ behavioral segments (Court Bookers, Members, VIP/Power User, Birthday Party Customers, Corporate, Lapsed, Lead, Never-Booked) are all missing.

---

## Profile properties (custom)

Sampled 15 profiles (5 by created, 10 by recently updated). Properties currently in use:

| Property | Sample value | Used by | Required for plan? | Severity |
|----------|--------------|---------|---------------------|----------|
| `Stripe Card Expiration Date` | "2030-04-30 04:56:00" | Stripe sync | Optional — not in plan | — |
| `Last Entry` | "2022-07-19 16:00:00" | Legacy custom | Maps loosely to `last_booking_date` | Replace with proper field |
| `Purchases` | "0" or "1" (string) | Legacy | Conceptually maps to `BOOKING_COUNT` (Flow 10) | Replace with numeric `booking_count` |
| `Money Spent` | "5.65" (string) | Legacy | Conceptually maps to `TOTAL_SPEND` (Flow 10) | Replace with numeric `total_spend` |
| `Service Type` | "Court A (Full Court)" | Legacy | Maps loosely to `last_sport_booked` | Replace; current value also references deprecated "Court A" naming |
| `Full Name` | string | Stripe | Use Klaviyo `first_name` / `last_name` instead | — |
| `Invoice` | string | Stripe | — | — |
| `$source` | "footer signup", "Multi-step email & SMS", numeric codes | System | OK | — |
| `$consent`, `$consent_timestamp`, `$consent_method`, `$consent_form_id` | system fields | Klaviyo Forms | Required for CASL; OK | — |
| `$phone_number_region` | "CA" | System | OK | — |
| Standard Klaviyo: `first_name`, `last_name`, `email`, `phone_number`, `location.city/region/country/timezone` | populated for some | System | OK | — |

### Properties REQUIRED for new flows but missing from Klaviyo

(All severity ranked HIGH unless noted. Names quoted from `klaviyo-master-buildsheet-v2.md` Section 02 + Section 04.)

**Flow infrastructure (sender / location):**
- `location_manager` (text) — used as `{{location_manager}}` in Flow 07 T01, Flow 09 T07; HIGH (blocks franchise-correct sender doctrine)
- `location_manager_email` (text) — used as `{{location_manager}}@therunrec.com`; HIGH
- `location_phone` (text) — used in Flow 09 T07 contact line; HIGH
- `location_name` (text) — used in Flow 07 T01 ("you're officially a `{{LOCATION_NAME}}` regular"); HIGH

**Booking metadata (Flows 02, 03, 06, 12):**
- `last_booking_date` (date) — drives Flow 03 trigger calculation, Flow 06 lapsed trigger (60 days since), Flow 12 abandoned booking; HIGH
- `last_sport_booked` (string) — drives Flow 02 personalization, segmentation; HIGH
- `booking_count` (numeric) — drives Flow 10 personalization (`{{BOOKING_COUNT}}`); HIGH
- `total_spend` (numeric, currency) — drives Flow 10 personalization (`{{TOTAL_SPEND}}`, `{{SAVINGS}}` math); HIGH
- `last_review_disposition` (string: positive/negative) — Flow 03 thumbs-up branching; MEDIUM

**Birthday (Flows 04, 05):**
- `dob` or `birth_date` (date, MM-DD format minimum) — drives Flow 04 trigger DOB-7 / DOB / DOB+1; HIGH
- `bday_gift_redeemed_year` (numeric) — Flow 04 redemption tracking, prevents duplicate gifting; MEDIUM
- `child_name` (string), `child_age` (numeric or birth date) — Flow 05 personalization fallback (the spec already notes these don't have to be clean); LOW (Flow 05 designed to work without these)
- `last_party_date` (date) — Flow 05 trigger (11 months after); HIGH for Birthday Customers list activation

**Membership / VIP (Flows 07, 10, 11):**
- `vip_since` (date) — Flow 07 trigger date stamp; HIGH
- `membership_tier` (string: "Common User"/"Tri-City Member") — Flow 11 routing; HIGH
- `membership_started_date` (date) — Flow 11 anniversary calc, Flow 11 1-year retention; HIGH
- `last_membership_renewal` (date) — Flow 11 lifecycle; MEDIUM
- `membership_status` (active/cancelled/lapsed) — Flow 11 win-back path; HIGH

**Corporate / B2B (Flow 09):**
- `corporate_inquiry_date` (date); HIGH
- `corporate_status` (active/parked/lost); HIGH
- `corporate_company_name` (string); MEDIUM
- `corporate_event_size` (numeric); LOW

---

## Flows (15 total — 8 LIVE, 7 DRAFT)

| Name | ID | Status | Trigger | Actions (sends + delays + branches) | Send mix | Maps to plan? |
|------|----|--------|---------|------------------|----------|----------------|
| 2025 - Welcome Flow (Visual Email) | RZsSVX | **live** | Added to List | 6 (3 emails, 1 boolean branch, 2 delays — ~60s + 24h) | 3 email | Plan Flow **01 Welcome** (partial — plan calls for 6 email + 1 SMS over 14 days; current is 3 email over ~24h) |
| 2025 - Welcome Flow (SMS Flow Part 1) | QZXWNA | **live** | Added to List (SMS Recipients) | 7 (4 SMS, 3 delays — 2d/2d weekend/7d) | 4 SMS | Plan Flow **01 Welcome SMS** (close enough — operator should confirm cadence vs plan's Day 14 SMS-grand-slam) |
| 2025 - Browse Abandoned (Visual Email) | RncQfn | **live** | Metric (likely Viewed Product) | 11 (4 emails, 3 boolean branches, 4 delays — all 1h) | 4 email | Closest to Plan Flow **12 Abandoned Booking** but actually triggered on `Viewed Product`, not on a real abandoned-booking event. Functionally mismatched. |
| 2025 - Browse Abandoned (SMS Part Flow 3) | WajxZ4 | **live** | Metric | 6 (3 SMS, 3 delays — 1h/24h/72h) | 3 SMS | SMS counterpart to above. Same caveat: not a real abandoned-booking trigger. |
| 2025 - Post Purchase (Visual Email) | VwYTDc | **live** | Metric (Successfully Paid) | 9 (5 emails, 1 boolean branch, 3 delays — 2d/2d/7d) | 5 email | Plan Flow **03 Post-Session + Review** (closest match) — but plan calls for trigger 2 hours after booking end-time, not on payment. Trigger needs reworking. |
| 2025 - Post Purchase (SMS Flow Part 4) | TXW8MN | **live** | Metric | 6 (3 SMS, 3 delays — 1h/2d/7d) | 3 SMS | SMS counterpart to post-purchase. |
| First Purchase Anniversary - Standard | UBQq2E | **live** | Metric (Successfully Paid + 365d delay) | 3 (1 update_customer write of `First Purchase`, 1 email, 1 365-day delay) | 1 email | Loosely overlaps Plan Flow **05 Birthday Party Anniversary** but is purchase-anniversary, not party-anniversary. Different intent. |
| New - Win Back (Email Flow Part 5) | YvvKXF | **live** | Metric | 7 (1 SMS, 3 emails, 3 delays — 20d/20d/30d) | 3 email + 1 SMS | Plan Flow **06 Win-Back (Lapsed)** — partial; plan calls for 4 email + 2 SMS over 31 days. Cadence is roughly aligned. |
| Draft - Abandoned Cart (Email Flow Part 2) | U25Ffk | draft | Metric | 3 (2 emails, 1 delay — 2d) | 2 email | Stub for cart-abandonment. Plan Flow **12** target. Never finished. |
| Draft - Abandoned Cart (SMS Flow Part 2) | R3f8r8 | draft | Metric | 1 (single SMS) | 1 SMS | Stub. |
| New - Browse Abandoned (Email Flow Part 3) | TTiUHs | draft | Metric | 9 (4 emails, 1 SMS, 4 delays/branches) | 4 email + 1 SMS | Older draft superseded by the live `RncQfn`. Archive candidate. |
| New - Post Purchase (Email Flow Part 4) | VvLxsN | draft | Metric | 7 (4 emails, 1 SMS, 2 delays) | 4 email + 1 SMS | Older draft superseded by live `VwYTDc`. Archive candidate. |
| Draft - Win Back (SMS Flow Part 5) | XHEScZ | draft | Metric | 1 (single SMS) | 1 SMS | Stub for SMS win-back. Plan Flow 06 SMS portion. |
| Essential Flow Recommendation_ | VxHPF8 | draft | **Unconfigured** | — | — | Klaviyo template suggestion, never built. Delete. |
| Essential Flow Recommendation_ | WH8EyY | draft | **Unconfigured** | — | — | Klaviyo template suggestion, never built. Delete. |

### Flows in plan but missing in Klaviyo

| # | Plan flow | Closest existing? | Gap |
|---|-----------|-------------------|-----|
| 02 | Booking Confirm + Reminders | None | No native booking-confirmation flow. Skedda likely sends its own — but no in-Klaviyo wiring. **Build from scratch.** |
| 03 | Post-Session + Review | "2025 - Post Purchase (Visual Email)" `VwYTDc` | Trigger needs change from `Successfully Paid` to "2 hours after booking end-time." Likely needs a custom event from Skedda. |
| 04 | Customer Birthday | None (Birthday Customers list `T8HDuV` has no flow) | Build from scratch. Requires `dob` profile property. |
| 05 | Birthday Party Anniversary | "First Purchase Anniversary - Standard" `UBQq2E` (only structurally similar) | Different intent. Build a new party-anniversary flow tied to Birthday Customers list with `last_party_date`. |
| 07 | VIP + Referral | None | No VIP/Power User segment exists. Need to build segment first, then flow. |
| 08 | Lead Nurture | None | No lead-nurture flow. Welcome Series serves as de-facto nurture for newsletter signups, but no separate funnel for unconverted leads at 30+ days. |
| 09 | Corporate & League Nurture | None | No B2B flow. No `corporate_inquiry` event captured. Form needed on website + Klaviyo Forms event. |
| 10 | Membership Conversion | None | No conversion flow. Trigger condition (5+ paid in 90 days at Common User threshold) needs the segment built. |
| 11 | Membership Lifecycle | None | No membership lifecycle. Stripe `customer.subscription.*` events not present in metric list — need Stripe→Klaviyo subscription event sync. |
| 12 | Abandoned Booking | "2025 - Browse Abandoned" pair (`RncQfn` + `WajxZ4`) | Mismatched trigger. Plan needs `booking_abandoned` custom event from Vercel checkout layer (per buildsheet, action item for Eyad/Rob). |
| 13 | Sunset / Hygiene | None | No deliverability cleanup flow. Critical gap — long-term sender reputation depends on this. |

### Flows in Klaviyo but NOT in plan (need decision)

| Flow | ID | Recommendation |
|------|----|---------------:|
| First Purchase Anniversary - Standard | UBQq2E | **Decision needed.** Single-email anniversary. Either rebuild as part of Flow 11 (Membership Lifecycle) or archive — purchase anniversary is not in the 13-flow plan. |
| Draft - Abandoned Cart (Email/SMS Part 2) | U25Ffk, R3f8r8 | **Archive.** Stubs from 2023, never finished. Will be replaced by Flow 12 build. |
| New - Browse Abandoned (Email Flow Part 3) | TTiUHs | **Archive.** Older draft superseded by the live `RncQfn`. |
| New - Post Purchase (Email Flow Part 4) | VvLxsN | **Archive.** Older draft superseded by the live `VwYTDc`. |
| Draft - Win Back (SMS Flow Part 5) | XHEScZ | **Archive.** Stub; will be folded into Flow 06 build. |
| Essential Flow Recommendation_ (x2) | VxHPF8, WH8EyY | **Delete.** Empty Klaviyo template suggestions, never configured. |

### Live flows that overlap the plan (decision: pause/replace vs. iterate)

The 6 LIVE flows below overlap parts of the plan but most are MISALIGNED on trigger or cadence. Build team must decide for each:

1. `RZsSVX` Welcome (Visual Email) — KEEP & EXTEND (it's the foundation of Plan Flow 01)
2. `QZXWNA` Welcome SMS — KEEP & RE-CADENCE (plan calls for 1 SMS at Day 14, current is 4 SMS over 11 days)
3. `RncQfn` + `WajxZ4` Browse Abandoned — **PAUSE** until real abandoned-booking event exists. Currently fires on `Viewed Product` which is high-volume / low-intent — risk of over-mailing.
4. `VwYTDc` + `TXW8MN` Post Purchase — KEEP but RE-TRIGGER on real session-end event when Skedda integration lands; meanwhile current `Successfully Paid` is the closest proxy.
5. `UBQq2E` First Purchase Anniversary — DECIDE (above).
6. `YvvKXF` Win Back — KEEP & EXTEND to match Flow 06 (4 email + 2 SMS over 31 days vs current 3 email + 1 SMS over 70 days).

---

## Templates (21)

All templates are `SYSTEM_DRAGGABLE` (Klaviyo's drag-drop editor). All but two were last edited in Jan 2023. The library is essentially the Klaviyo default starter pack.

| Name | ID | Last Edited | Recommendation |
|------|----|------------|----------------|
| (customer) Approved Appointment | SjQEie | 2022-12-04 | Archive — older booking confirm template, predates current Skedda flow |
| 2022-11-08 11:48 1 Column | TKddT9 | 2022-11-09 | Archive — autosave |
| 24 Hour Access | TKZSi5 | 2022-12-15 | Review then archive — likely old member access template |
| Adjustable Template (Rammah) | RrVxqM | 2023-05-23 | Archive — personal template by ex-team |
| Customer winback 1 | Xci8zH | 2022-11-03 | Archive — predates current win-back flow |
| Fast Pass | WLZMLG | 2023-04-23 | Archive |
| Holiday sale | UW7Va7 | 2023-01-30 | Archive |
| Newsletter #1-#8 (8 templates) | VPKUQQ, UPewjQ, Wn9H85, UcdtHd, VmCiB5, S5DD96, Tf4u2W, TMsgib | 2023-01-30 | All Klaviyo defaults. Keep one as a reference; archive the rest. |
| Nike Campaign / Reminder | VaCeSq, X2nHNf | 2022-12 | Archive — old promo |
| Nonprofit - Event/Multiple Updates/Newsletter/Volunteer | TTngRi, VFNamM, WRcZfD, S2j8Zr | 2022-07 | Archive — wrong vertical, Klaviyo defaults |

**Critical gap:** There are **NO branded RunRec base templates** in the library. The plan's "Template architecture" calls for three base templates (Flow/Personal minimalist, Birthday Party branded, VIP/Status premium dark). All three need to be built from scratch.

The current live flows (`RZsSVX`, `RncQfn`, `VwYTDc`, `YvvKXF`) reference templates inline — those templates are NOT in this list because they were authored inside the flow editor as flow-messages, not promoted to the global Templates library. Build team should consider promoting key designs to the global library so the 13 new flows can reuse them.

---

## Tags

**0 tags exist.** No tag taxonomy. Plan calls for tags like `flow_05_anniversary_year_{{YEAR}}`, channel/funnel tagging on flows, and segment tagging. **Tag system needs to be built from zero.**

Recommended starter tag groups (for the build team to create):
- **Flow purpose** (acquisition, retention, transactional, recovery, b2b, hygiene)
- **Customer type** (court_booker, member, birthday, corporate, lead, vip)
- **Channel** (email, sms, both)
- **Year** (2026, 2027 — for time-bound campaigns)

---

## Metrics / Events being captured

**58 metrics** registered. Below are the ones that matter for the plan:

| Event | Source | Used by | In plan? |
|-------|--------|---------|----------|
| Successfully Paid | Stripe | Currently triggers Post Purchase, Anniversary flows | Plan equivalent: `booking_completed` (real-time, post-session) |
| Failed Payment | Stripe | None (could trigger payment-recovery sub-flow) | Optional — not in 13 |
| Refunded Payment | Stripe | None | Optional |
| Issued Invoice | Stripe | None | Could feed Flow 11 lifecycle |
| Subscribed to Email Marketing | Klaviyo | None — could trigger Welcome | Plan: yes, Flow 01 trigger |
| Subscribed to SMS Marketing | Klaviyo | None | Plan: Flow 01 SMS branch |
| Form submitted by profile | API + Klaviyo | None | Plan: feeds Flow 08 Lead Nurture, Flow 09 Corporate |
| Active on Site | API (Klaviyo JS) | None | Plan: feeds Flow 12 + segmentation |
| Viewed Product | API (Klaviyo JS) | Browse Abandoned flows | Plan: weak signal; Flow 12 needs `booking_abandoned` instead |
| Placed Order | Shopify | None (zero properties — likely not firing) | Plan: not used; RunRec is service-based, not product-based |

### Events REQUIRED for new flows but not yet captured

(All HIGH severity unless noted.)

- `booking_created` — fires when a customer books a session via Skedda. Required for **Flow 02 Booking Confirm**. Missing.
- `booking_completed` (or "session ended") — fires 2 hours after session end-time. Required for **Flow 03 Post-Session + Review**. Missing — currently using `Successfully Paid` as a proxy which is wrong (paid != attended).
- `booking_abandoned` — fires when a customer reaches Vercel checkout but doesn't complete payment within 30min. Required for **Flow 12 Abandoned Booking**. Missing — buildsheet flags this as Eyad/Rob action item.
- `corporate_inquiry` — fires when website B2B contact form is submitted. Required for **Flow 09 Corporate**. Missing — needs website form + Klaviyo event sync.
- `membership_started` (Stripe `customer.subscription.created`) — required for **Flow 11 Membership Lifecycle**. Missing — Stripe is connected but only Payment events flow, not Subscription events.
- `membership_cancelled` (Stripe `customer.subscription.deleted`) — required for **Flow 11 Win-Back path**. Missing.
- `birthday_party_completed` — fires day-of after a birthday party. Required for **Flow 05 Anniversary** trigger (11mo later). Missing — likely manual data entry today.
- `nps_thumbs_up` / `nps_thumbs_down` (custom event from Klaviyo email click or external survey) — required for **Flow 03 T01 NPS check** branching. Missing.

### Other notable events (probable data quality issues)

- Two `Active on Site` metrics exist (`Sz46Tw` from 2022, `YeRy7w` from 2026). Suggests JS snippet was reinstalled at some point. Build team should verify which one is current and which has historical data.
- Two `Form submitted by profile` and two `Form completed by profile` metrics (one from 2024, one from 2026). Same duplication issue. Likely Klaviyo Forms migration.

---

## Implementation gaps (the actual work)

### Profile properties to create (Settings → Profile properties)
1. `location_manager`, `location_manager_email`, `location_phone`, `location_name` (text) — franchise sender doctrine
2. `last_booking_date` (date), `last_sport_booked` (string), `booking_count` (numeric), `total_spend` (numeric)
3. `dob` (date), `bday_gift_redeemed_year` (numeric), `last_party_date` (date)
4. `vip_since` (date), `membership_tier` (string), `membership_started_date` (date), `last_membership_renewal` (date), `membership_status` (string)
5. `corporate_inquiry_date` (date), `corporate_status` (string)
6. `last_review_disposition` (string)

### Lists to create / archive
- **Archive:** Camp Waitlist (Summer 2025) `ULfg93` — past event, no future use
- **Keep & wire to flows:** Birthday Customers `T8HDuV` (Flow 04, 05), Tri-City Member Emails `XbJSh7` (Flow 11), Skedda Email List `WyA26L` (Flow 06, Flow 07 segment source), Previous Campers `RVpxwS` (Flow 06), Principle Emails `SQQBdj` (likely Flow 09), PlayForever Signups `TS6RMf` (Flow 01 variant or its own welcome), PlayForever League `XHBsJD` (Flow 09 variant)

### Segments to create
1. **Engagement segments** (5 from CMO skill): 30-Day Engaged, 60-Day Engaged, 90-Day Engaged, Unengaged 90+, (New Subscribers exists)
2. **Behavioral segments** (8 from plan): Court Bookers (paid in last 90d), Members (membership_status = active), VIP/Power User (5+ Successfully Paid in 90d), Birthday Party Customers (in T8HDuV list), Corporate (corporate_status = active), Lapsed (60+ days since last_booking_date), Lead (in Newsletter list, never paid), Never-Booked (in Newsletter list, no Successfully Paid event ever)

### Custom events to wire
1. `booking_abandoned` from Vercel checkout (Eyad + Rob action — already flagged in buildsheet)
2. `booking_created`, `booking_completed`, `birthday_party_completed` from Skedda or operator workflow
3. `corporate_inquiry` from website form
4. Stripe Subscription events (`subscription.created`, `subscription.cancelled`, `invoice.payment_succeeded`) — verify Stripe→Klaviyo integration scope, may need to extend the existing connection

### Templates to build
1. Base **Flow / Personal** template — minimalist HTML, single column, logo + 1-2 links (used by Flows 01, 03, 06, 08, 09)
2. Base **Birthday Party** template — branded, party photos (Flows 04, 05)
3. Base **VIP / Status** template — premium dark (Flow 07 only)

### Tags taxonomy to create
- Tag groups + initial tags per the recommendation above

### Flows to build (per plan-Section 04)
- 12 new flows (Flow 02 + Flows 04-13). Flow 01 exists and needs extension.

### Existing flows to clean up
- **Archive:** 5 draft stubs (`U25Ffk`, `R3f8r8`, `TTiUHs`, `VvLxsN`, `XHEScZ`) and 2 unconfigured "Essential Flow Recommendation_" (`VxHPF8`, `WH8EyY`)
- **Pause:** Browse Abandoned pair (`RncQfn`, `WajxZ4`) until real `booking_abandoned` event exists — current `Viewed Product` trigger is too noisy and may be hurting deliverability
- **Decide:** First Purchase Anniversary `UBQq2E` (rebuild into Flow 11 or archive)

---

## Risks / cautions for the build team

### HIGH

1. **Browse Abandoned flows are firing on `Viewed Product`, not real intent.** The two live `RncQfn` (4 emails) + `WajxZ4` (3 SMS) flows trigger every time someone views a product on the site. Combined with high-volume Klaviyo JS tracking, this could be sending 7 messages per browser session in the worst case. Pause these before building Flow 12 to avoid colliding with the new flow AND to investigate whether the existing flow has been over-mailing since launch.

2. **Sender domain authentication status unknown.** MCP cannot verify DKIM/SPF/DMARC + custom sending domain. Operator must confirm in Klaviyo UI. If not configured, deliverability will sink as soon as flow volume increases. Plan calls for this to be confirmed in Section 02 "Authentication / deliverability."

3. **No sunset / hygiene flow exists (Flow 13).** With 2,989 marketable profiles and a long-running list (oldest profiles from 2022), there is almost certainly a population of dead addresses inflating the list. Sending the planned 12 new flows to this list without a sunset flow will erode sender reputation. Flow 13 should be built first, or in parallel with Flow 01.

4. **Welcome Flow (live) has no purchase-status branch.** The plan's Welcome Series and the d2c best-practice both call for "split by purchase status." Current `RZsSVX` has a boolean branch but it's not labeled in the read; build team should inspect whether the branch is doing the right work. If not, the bookers on the Skedda Email List (767 profiles) are getting a generic Welcome instead of a converted-customer track.

5. **Two duplicate metrics (`Active on Site`, `Form submitted by profile`).** Could mean two Klaviyo JS snippets are loaded on the site, or that an integration was reconnected without cleanup. Build team should confirm which is current — if both are firing, segment definitions tied to the wrong one will silently miss data.

### MEDIUM

6. **Stripe integration is paid-events-only.** Subscription lifecycle (Membership Conversion → Lifecycle in Flows 10/11) needs `customer.subscription.*` events. Verify the existing Stripe→Klaviyo integration includes these or extend it. Same for `invoice.payment_failed` if Flow 11 needs dunning.

7. **Newsletter list is single-opt-in.** Not a CASL violation per se (existing relationship covers it for legacy contacts), but new signups should move to double opt-in to align with the rest of the lists and reduce bot/spam-trap risk.

8. **No tag taxonomy exists.** All 13 flows in the plan reference tagging (e.g., `flow_05_anniversary_year_{{YEAR}}`). Build team must create the tag system before flows go live, or attribution and reporting will be impossible.

9. **First Purchase Anniversary `UBQq2E` is a 365-day delay flow that has been live since 2022.** Anyone who paid in the last 4 years is in this flow's queue. If it's archived, those queued sends die. If it's kept, it overlaps with Flow 05 and Flow 11. Operator should make the call before any flow archival.

### LOW (informational)

10. **Birthday Customers list (`T8HDuV`, 19 profiles) has zero automation.** This is the entire planned-Flow-04+05 audience and is currently dormant. As soon as Flow 04 and Flow 05 are built, those 19 profiles will receive their first contact in months — operator should write a "we're back" intro campaign before flows go live to avoid surprise.

11. **Skedda Email List (`WyA26L`, 767 profiles, double opt-in)** is the largest pool of paying customers and has zero automation. This is the highest-value list to wire flows to.

12. **All existing templates are essentially default Klaviyo starter templates** — none are RunRec-branded. Confirms the Templates section gap.

---

## Quick stats summary

| Asset class | Existing | Needed by plan | Gap |
|-------------|----------|----------------|-----|
| Lists | 12 | 12+ (most exist) | Mostly covered |
| Segments | 2 | 13+ (5 engagement + 8 behavioral) | -11 |
| Profile properties (custom for plan) | ~5 (legacy strings) | ~22 | -17 |
| Flows (LIVE) | 8 | 13 | -5 net (more like -10 once misalignments accounted for) |
| Flows (DRAFT, useless stubs) | 7 | 0 | Archive 7 |
| Templates (branded base) | 0 | 3 | -3 |
| Tags | 0 | ~15-20 | -all |
| Custom events | 0 | 8 | -8 |
| Stripe sub-events wired | 0 | 3 | -3 |
