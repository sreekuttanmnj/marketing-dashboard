# Looker Studio Build Guide — Marketing KPI Dashboard

**Audience:** The teammate who executes this in Looker Studio.
**Time to build:** ~30-45 minutes.
**Output:** A single Looker Studio report with auto-refreshing GA4 + Search Console data, plus a Google Sheet for the product-database KPIs.
**Scope:** OnlineCheckWriter + ZilMoney, US traffic only.

---

## Step 1 — Upload the internal-KPI template to Google Drive (5 min)

1. Open https://drive.google.com
2. Upload `data/internal_kpi_template.csv` from this folder.
3. Right-click the uploaded file → **Open with → Google Sheets** (this converts CSV to a native Sheet so Looker Studio can read it live).
4. Rename the sheet to **"Marketing KPIs — Internal Database"**.
5. Share it with everyone on the marketing team who will fill it in. Populate the empty cells from the product/billing database. The columns are pre-named to match the KPI requirements.

**Column reference (already in the sheet):**

| Column | Source | Definition |
|---|---|---|
| `trial_signups` | Product DB | Non-paid users who registered, US only |
| `paid_signups_checks` | Billing DB | New paid customers on a check plan, US only |
| `paid_signups_digital` | Billing DB | New paid customers on a digital plan, US only |
| `kyc_submitted` | Onboarding DB | Customers who started KYB/KYC, US only |
| `kyc_approved` | Onboarding DB | Customers who completed KYB/KYC successfully |
| `kyc_approval_pct` | Computed | `kyc_approved / kyc_submitted` — Looker Studio will calculate this |
| `paying_customers_checks` | Billing DB | Active paying customers (check plans), US only |
| `paying_customers_digital` | Billing DB | Active paying customers (digital plans), US only |

---

## Step 2 — Create the Looker Studio report (5 min)

1. Open https://lookerstudio.google.com
2. Click **+ Create → Report**
3. You'll be prompted to add a data source. Add the ones below.

---

## Step 3 — Add data sources (10 min)

### Data source 1: GA4 — OnlineCheckWriter

1. **Add data → Google Analytics**
2. Choose account **Onlinecheckwriter (Master)** → property **Online Check Writer (main)** (ID `254458280`)
3. Click **Add**
4. Rename the source to **"GA4 — OCW"** (sidebar → Resource → Manage added data sources)

### Data source 2: GA4 — ZilMoney

Repeat with account **Zil Money** → property **zilmoney.com - GA4** (ID `311648623` — the one with traffic; ignore the empty 391616687).
Rename to **"GA4 — ZilMoney"**.

### Data source 3: Search Console — OCW

1. **Add data → Search Console**
2. Site: `https://onlinecheckwriter.com/`
3. Table type: **Site Impression**
4. Rename to **"GSC — OCW"**.

### Data source 4: Search Console — ZilMoney

Same as above, site `https://zilmoney.com/`. Rename to **"GSC — ZilMoney"**.

### Data source 5: Internal KPIs Sheet

1. **Add data → Google Sheets**
2. Select the **"Marketing KPIs — Internal Database"** sheet from Step 1.
3. Use the first row as headers ✓
4. Rename to **"Internal KPIs"**.

---

## Step 4 — Apply the US filter to all GA4 / Search Console sources (5 min)

Looker Studio applies filters at the chart level OR the page level. Page-level is faster.

1. **Resource → Manage filters → + Add filter**
2. Name it **"US Only — GA4"**
3. Set: `Include` · `Country` · `Equal to (=)` · `United States`
4. Save.
5. Repeat for Search Console with: `Include` · `Country` · `Equal to (=)` · `usa` (lowercase ISO-3 in GSC).

Then on each page, in the **Page settings → Page filters**, attach the appropriate filter so it applies to every chart on that page.

---

## Step 5 — Add the calculated fields (5 min)

In each data source, click the pencil icon and add these fields:

### In each GA4 source

- **Field name:** `Week Label`
  **Type:** Calculated · Formula: `CONCAT("W", FORMAT_DATETIME("%V", Date))`
  *Gives clean labels like W13, W14...*

- **Field name:** `Brand`
  **Type:** Calculated · Formula: `"OnlineCheckWriter"` (use the matching brand string per source)
  *Lets you union sources together later.*

### In the Internal KPIs source

- **Field name:** `KYC Approval %`
  **Formula:** `kyc_approved / kyc_submitted`
  **Format:** Percent

---

## Step 6 — Build the report pages (15 min)

### Page 1 — Headline

- **Scorecards** (across the top): Sessions (last 7d), Users, Sessions WoW %, Search Console Clicks.
- For Sessions WoW %, use the comparison-date feature: set the date range to "Last 7 days" with comparison "Previous period" — Looker Studio computes the delta natively.
- **Time series chart** (large, below the scorecards): X = Date (by week), Y = Sessions, breakdown dimension = data source (OCW / ZilMoney). Use the blend feature (see Step 7).

### Page 2 — By Brand

For each brand (one row per brand), put:
- A small time-series chart: Sessions + New Users
- A table: Week, Sessions, Users, New Users, Search Clicks, Search Impressions
  - To combine GA4 sessions with Search Console clicks in one table, use a **data blend** (Step 7).
- A bar chart: Channel breakdown (`sessionDefaultChannelGroup`)

### Page 3 — Conversions (GA4 events)

- **OCW**: Time-series of `live-sign-up`, `live-upgrade-plan`, `live-pay-card`. Filter event_name to those three.
- **ZilMoney**: Time-series of `contact_button_click`, `generate_lead`. Add a banner above noting "no reliable signup event tracked — see notes".

### Page 4 — Internal-Database KPIs

- Bind all tables/charts on this page to the **Internal KPIs** data source.
- Tables: one per brand, showing the full row from the sheet.
- Scorecard: KYC Approval % (latest week, with WoW comparison).
- Pivot chart: Trial signups vs Paid signups by week, stacked.

---

## Step 7 — Blending GA4 and Search Console for the same brand (optional but recommended)

Looker Studio blends let you join GA4 sessions with Search Console clicks by week so they show in one table.

1. **Resource → Manage blends → + Add a blend**
2. Add the GA4 source on the left, Search Console source on the right.
3. Join key: `Date` (both have it). Pick **Left outer** so GA4 weeks without Search Console rows still show.
4. Choose metrics: from GA4 — Sessions, Users, New Users. From Search Console — Clicks, Impressions.
5. Save with a clear name like **"OCW blended"**.

Build one blend per brand. Charts can then pull from the blend instead of either raw source.

---

## Step 8 — Share

1. Click **Share** (top right).
2. Add the relevant stakeholders.
3. Choose **Viewer** permission for them — keeps the build read-only for non-editors.

---

## Maintenance

- **Looker Studio auto-refreshes** GA4 and Search Console data with each page load — no work needed.
- **The Internal KPIs sheet** is the only thing the marketing team has to update each week. New rows added to the bottom flow through automatically.
- **Week-over-week** comparisons work via Looker Studio's built-in date-comparison feature — enable it once per chart.

## Reusable URL when done

When the report is built, share its URL like:
`https://lookerstudio.google.com/reporting/<report-id>`

That URL becomes the canonical source for the weekly review.
