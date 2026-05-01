# Decisions Pending — Operator Action Items

**For:** Sal
**Purpose:** Single tracker for all open decisions that block the plug-and-play package. Each decision: what's needed, default recommendation, deadline, who's blocked.

When you make a decision, mark it ✅ DECIDED and add the choice + date.

---

## Critical decisions (block launch — resolve THIS WEEK)

### 1. Skedda → Klaviyo integration approach

**Why it matters:** Blocks Flows 02, 03, 12 entirely. Most of the dashboard depends on Skedda firing booking events into Klaviyo.

**Choice required:**
- A. Zapier/Make.com middleware (~$30-100/mo, fast setup, stable)
- B. Custom Vercel/Cloudflare Workers middleware (engineering time, cheap, full control)
- C. Direct webhook from Skedda (depends on Skedda's webhook capabilities)

**Recommendation:** A (Zapier) for v1, migrate to B at franchise scale. Get the flows live first.

**Deadline:** Before Eyad/Rob start engineering tickets 1-3.
**Blocked:** Engineering backlog (`09-engineering-backlog.md`).
**Status:** ⬜ pending

---

### 2. Custom profile properties — operator confirms list

**Why it matters:** 17+ custom properties needed. Some require operational context Sal owns.

**Specifically need confirmation on:**
- `location_manager` — set default for Waterloo HQ to "Sal" or "Sal Dader" or something else?
- `location_manager_email` — `sal@therunrec.com` confirmed?
- `location_phone` — `519-800-9894` confirmed?
- `membership_tier` values — what are the actual tier names? "None / Member / VIP" or different?
- `corporate_status` values — "Lead / Active / Lapsed" or different?

**Recommendation:** Approve the defaults above. Adjust later if franchise locations introduce variations.

**Deadline:** Before admin starts pre-flight setup Section D.
**Blocked:** Pre-flight setup, all per-flow guides that reference these properties.
**Status:** ⬜ pending

---

### 3. Voice/signature inconsistencies (the agents flagged 5 places)

**Why it matters:** Multiple agents found mismatches between dashboard HTML and buildsheet. Without resolution, the per-flow guides will conflict.

**Specifically:**

| Where | Dashboard says | Buildsheet says | Recommendation |
|-------|---------------|-----------------|----------------|
| Flow 08 T02 sig | "Izzy, RunRec" | "Sal" | Use "The RunRec Team" — fits voice doctrine for informational/segmenting touch |
| Flow 09 T04 sig | "Izzy, RunRec" | "Sal" | "The RunRec Team" — informational tier presentation |
| Flow 09 T05 SMS body | "-Izzy" | "-Sal" | "-Sal – RunRec Co-Owner" — matches sender label change per spec |
| Flow 09 T06 sig | "Izzy, RunRec" | "RunRec Team" | "The RunRec Team" — matches buildsheet, voice doctrine |
| Flow 11 T01 sig | "Izzy, RunRec" | (Sal & Izzy removed per spec) | "— The RunRec Team" — matches voice doctrine |

**Recommendation:** Adopt the recommendations above (right column). Sweep the dashboard HTML to match. Sweep the per-flow guides to match.

**Deadline:** Before admin builds Flows 08, 09, 11.
**Blocked:** Flows 08, 09, 11 implementation accuracy.
**Status:** ⬜ pending

---

### 4. Section 9 sender direction approach

**Why it matters:** All 5 Section 9 emails use `{{location_manager}}@therunrec.com`. There are two ways to make this work in Klaviyo:

- **Option A:** Single fallback sender + merge tag in From: name only. Faster setup, less correct for multi-location.
- **Option B:** Separate sender per location, dispatched via conditional flow split. More setup, multi-location-correct.

**Recommendation:** Option A for Waterloo-only launch (this month). Option B at franchise scale (when Burlington opens).

**Deadline:** Before admin builds Flow 09.
**Blocked:** Flow 09 implementation.
**Status:** ⬜ pending

---

### 5. Code-vs-reply mechanic — final exception list

**Why it matters:** Mechanic doctrine says "codes are out." But the audit flagged a few touches where the spec said "no adjustments" (which means the original code may still apply). Clarify whether:

- All codes are universally out (recommendation per buildsheet 00.6)
- OR some flows retain codes as documented exceptions

**Recommendation:** All codes out. No exceptions. Reply mechanic across the board. The CMO already swept this in the dashboard.

**Deadline:** Confirm before admin builds reply-mechanic flows (04, 06, 08, 09, 10, 11, 12).
**Blocked:** Flows 04, 06, 08 + downstream.
**Status:** ⬜ pending — recommend approve the recommendation

---

## Important decisions (resolve before specific flows go live)

### 6. Inbox triage SLA — who answers replies?

**Why it matters:** Flows 04, 06, 08, 09, 10, 11, 12 all use reply-to-redeem mechanic. If no one answers replies within hours, customers get pissed.

**Specifically need:**
- Who monitors `contact@therunrec.com` and the SMS reply inbox?
- What's the target SLA? (Recommend: 4 hours business hours, 24 hours weekends)
- What happens if the answerer is on vacation?

**Recommendation:** Eyad as primary, Sal as backup. 4-hour business hours target. Vacation = auto-reply with expected response time.

**Deadline:** Before any reply-mechanic flow goes live.
**Blocked:** Flows 04, 06, 08, 09, 10, 11, 12 activation.
**Status:** ⬜ pending

---

### 7. Privacy policy — does RunRec have one?

**Why it matters:** CASL/GDPR/CAN-SPAM all require a working privacy policy linked from emails. If you don't have one, you're exposed.

**Action:**
- Verify whether `therunrec.com/privacy` (or similar) exists
- If not: commission a template-based policy now (Termly, iubenda, etc. — $50-200)
- Link from email footer (Klaviyo template)

**Deadline:** Before any flow goes live to general audience.
**Blocked:** All flows.
**Status:** ⬜ pending

---

### 8. Sender reputation warm-up plan

**Why it matters:** If you're using a brand-new sending domain (`send.therunrec.com`), Klaviyo recommends 2-4 weeks of gradual volume ramp before high-volume sends.

**Choice:**
- A. Use existing `therunrec.com` directly (already warm if you've sent before)
- B. Set up new `send.therunrec.com` and warm up over 4 weeks
- C. Use Klaviyo's shared IP (acceptable for low volume)

**Recommendation:** B for franchise scale, but skip for initial 13-flow launch — use whatever's already authenticated.

**Deadline:** Before Flow 01 (Welcome) goes live to full Newsletter list (1,435 profiles).
**Blocked:** Flow 01 launch at scale.
**Status:** ⬜ pending

---

### 9. List re-consent for CASL implied-consent expiry

**Why it matters:** CASL implied consent expires 24 months after the relationship ends. The Skedda Email List (4-year-old) and Previous Campers list have profiles past this window. Sending to them is a CASL violation risk.

**Action:**
- Audit which profiles are past the 24-month window
- Either: (A) re-consent campaign (single email asking "still want to hear from us?"), or (B) suppress these profiles from marketing flows

**Recommendation:** A — run a re-consent campaign before Flow 06 (Win-Back) launches. Suppress non-responders.

**Deadline:** Before Flow 06 launch.
**Blocked:** Flow 06.
**Status:** ⬜ pending

---

### 10. Test profile setup mechanics

**Why it matters:** Test protocol (`04-test-protocol.md`) needs 3 dedicated test profiles with real-looking email addresses.

**Choice:**
- A. DNS forward — set up `test-active@therunrec.com` etc. as forwards to Sal's inbox (requires Rob)
- B. Gmail plus-addressing — `sal+test-active@gmail.com` etc. (faster, less brand-realistic)

**Recommendation:** B for v1 (1 hour setup), A for v2 (when franchise scale needs branded test addresses).

**Deadline:** Before admin runs test protocol on first flow.
**Blocked:** Test protocol execution.
**Status:** ⬜ pending

---

## Lower-priority decisions (resolve over coming weeks)

### 11. First Purchase Anniversary flow (existing `UBQq2E`)

**Why it matters:** Inventory found this existing flow. It overlaps with Flow 05 (Birthday Party Anniversary). Decision: keep, archive, or replace?

**Recommendation:** Archive `UBQq2E` once Flow 05 is built and tested.

**Deadline:** Before Flow 05 launch.
**Status:** ⬜ pending

---

### 12. Templates: build from scratch or commission designer?

**Why it matters:** 3 base templates needed (Flow/Personal, Birthday Party, VIP/Status). Currently 0 RunRec-branded templates exist.

**Choice:**
- A. Admin builds in Klaviyo from scratch (free, time-consuming, may look generic)
- B. Hire designer to build branded templates ($500-1500, 1-2 weeks)
- C. Use Klaviyo's modern template library + customize

**Recommendation:** C for v1 (admin can do this in 4 hours), B for v2 if open rates lag.

**Deadline:** Before Flow 01 launch.
**Status:** ⬜ pending

---

### 13. Sunset flow segment definition

**Why it matters:** Flow 13 (Sunset) auto-suppresses unengaged profiles. The criteria for "unengaged" needs to be precise.

**Recommendation:** "No email open OR click in last 365 days AND no booking in last 365 days." Sal previews segment population before activation — expect 900-1,500 profiles flagged on first run.

**Deadline:** Before Flow 13 activation.
**Status:** ⬜ pending

---

## Decision log (mark when resolved)

When you decide on something, append a row here:

| Date | Decision # | Choice made | Notes |
|------|------------|-------------|-------|
| | | | |

---

## Quick-fire decision template

If you want to decide all 13 in one pass: open this doc, walk top to bottom, write your choice in 30 seconds each. Total time: 10-15 minutes. Then the admin can move forward without blockers.

If you want to defer any: mark it ⏸ with a date you'll revisit. The admin will work around deferred items where possible.
