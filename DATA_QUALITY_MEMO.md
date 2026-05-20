# Data Quality Memo — Marketing KPI Dashboard

**Date:** 2026-05-20
**Pull window:** 2026-03-23 to 2026-05-17 (8 ISO weeks, US traffic)
**Scope:** OnlineCheckWriter, ZilMoney
**For:** Marketing team review, ahead of the weekly KPI meeting

This memo lists the data-quality issues found while building the dashboard, ranked by how much they affect the KPIs in scope.

---

## Tracking gaps (affect dashboard accuracy)

### 1. ZilMoney signup tracking is effectively absent

- **Symptom:** The dedicated signup event (`singup_zm` — note the misspelling in the event name itself) fires fewer than 1 time per week US. `generate_lead` fires ~30/week. There is no equivalent of OCW's `live-sign-up`.
- **Impact:** The "non-paid users on trial" KPI for ZilMoney cannot be computed from GA4. The product database is the only source.
- **Action:** Either (a) instrument a proper `sign_up` event on the ZilMoney signup completion page (preferred — gives marketing real-time visibility), or (b) commit to pulling signup counts from the product database and accept that GA4 will not have this KPI for ZilMoney.
- **Note:** While instrumenting, also fix the event-name typo — `singup_zm` should be `signup_zm`.

### 2. Duplicate ZilMoney GA4 property

- **Issue:** Two properties named "zilmoney.com - GA4" exist:
  - `properties/311648623` — active, ~2,800 US sessions/week
  - `properties/391616687` — empty, never received data
- **Risk:** Anyone setting up a new tool (Looker Studio, an integration, an MCP) might pick the wrong one and silently get zero data.
- **Action:** Delete or clearly rename `391616687` (e.g., "ARCHIVED — DO NOT USE"). The Looker Studio build guide explicitly points to 311648623.

---

## Interpretation notes (affect how to read the numbers)

### 3. OCW sessions are heavily weighted toward existing-user logins

- **What this means:** The `/login` page accounts for ~50%+ of OCW sessions. Most "traffic" to OCW is existing customers signing in, not new acquisition.
- **Implication:** When discussing acquisition, prefer:
  - `New Users` column (≈6,700/week recently), or
  - `live-sign-up` event count (≈1,050-1,300/week)
- Both are surfaced on the dashboard. "Total Sessions" stays in the headline for total-traffic comparisons.

### 4. High "Unassigned" channel volume on OCW (week 13 only)

- **W13** OCW: 2,226 Unassigned sessions out of 25,096 (8.9%).
- **W14–W20**: Unassigned drops to 300–500/week (1–2%).
- **Likely cause:** A campaign or referral source briefly broke its UTM tagging in late March.
- **Action:** Spot-check the W13 Unassigned traffic source/medium in GA4 to confirm there isn't a recurring tagging gap. Low priority — it self-corrected.

### 5. CTR drift on OCW search results

- CTR moved from 2.16% (W17) to 1.35% (W20). Position got worse (15 → 17).
- Worth a separate SEO investigation. Outside the scope of this dashboard but worth flagging during the review.

---

## Methodology

- **US filter on GA4** uses the `country` dimension, derived from IP geolocation. ~99% accurate; minor misclassification on VPN/CGNAT traffic is normal.
- **US filter on Search Console** uses Google's own country signal (search-side, not IP). Slightly more conservative than GA4's US count.
- GA4 sessions and Search Console clicks will not reconcile 1:1 — they measure different things. Search Console counts only clicks from Google search results; GA4 counts all session starts.
- **Week 21 (May 18 onward) is excluded.** It only had ~2 days of data when pulled, so it would mislead the week-over-week trend. The dashboard ends at W20 (May 11–17).
- **Timezones:** Both OnlineCheckWriter and ZilMoney report in America/Chicago, so weekly buckets line up cleanly across the two brands.

---

## Recommended sequence for the meeting

1. Walk through the headline KPIs and trend chart.
2. Brand-by-brand: traffic, channel mix, conversion events.
3. Internal-database KPI section — marketing team commits to filling it in next cycle.
4. Open the notes at the bottom: tracking gaps + interpretation context.
5. Decide what to instrument or fix before next week's report.

If the ZilMoney signup instrumentation lands by next cycle, the dashboard becomes materially more useful for that brand. Until then, treat GA4-derived ZilMoney signup numbers as directional and the product database as the source of truth.
