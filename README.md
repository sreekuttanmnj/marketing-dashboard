# Marketing KPI Dashboard

Built 2026-05-20 in support of the weekly marketing KPI review.
Covers 8 ISO weeks ending Sunday May 17, 2026. US traffic only. Brands in scope: **OnlineCheckWriter** and **ZilMoney**.

## What's here

| File | What it is |
|---|---|
| `dashboard.html` | Open in any browser. Ship-ready light-theme dashboard with all GA4 + Search Console data baked in. Use this if the Looker Studio build slips before the meeting. |
| `LOOKER_STUDIO_BUILD_GUIDE.md` | Step-by-step instructions for the marketing team to build the equivalent dashboard in Looker Studio (~30-45 min). |
| `DATA_QUALITY_MEMO.md` | Tracking caveats and methodology notes. Read before the meeting. |
| `data/ga4_us_traffic_weekly.csv` | Weekly US sessions, users, new users — OCW + ZilMoney × 8 weeks. |
| `data/ga4_conversions_weekly.csv` | Weekly signup / conversion events from GA4. |
| `data/gsc_us_weekly.csv` | Weekly US clicks + impressions from Search Console. |
| `data/internal_kpi_template.csv` | Empty template for the product-database KPIs. Upload to Google Drive, convert to Sheet, marketing team fills it in. |

## KPI coverage

| KPI | Status | Source |
|---|---|---|
| US website traffic (weekly) | Covered | GA4 — `ga4_us_traffic_weekly.csv` |
| Search clicks + impressions (weekly) | Covered | Search Console — `gsc_us_weekly.csv` |
| Trial signups (OCW only) | Covered | GA4 `live-sign-up` event |
| Paid signups (checks vs digital) | Pending | Product database — internal sheet |
| KYB/KYC submitted / approved / approval % | Pending | Product database — internal sheet |
| Paying customers (checks vs digital) | Pending | Product database — internal sheet |

## Tracking caveats (also documented at the bottom of the dashboard)

- **ZilMoney**: the dedicated signup event fires <1 time per week. The dashboard uses `contact_button_click` and `generate_lead` as lead proxies. The product database is authoritative for ZilMoney signups.
- **OnlineCheckWriter**: the `/login` page drives ~50% of sessions. When discussing new-customer acquisition, prefer `New Users` or the `live-sign-up` event over raw Sessions.

## To re-run with fresh data

Open Claude Code in this folder and ask for "fresh marketing dashboard pull." All queries are reproducible — no manual pipeline, no API keys to manage. The pull window slides forward automatically.
