# User Guide: Noodles Crypto Social Analytics Dashboards
**Author:** Pavendhar Tamilarasu

## Overview

This guide explains how to open and use the two Power BI dashboards built
for this project. They show how much attention different cryptocurrencies
get on Twitter and Reddit — which currencies are talked about most, how
engagement is trending over time, and how Twitter compares to Reddit.

**Important scope note:** these dashboards report on **social media
engagement** (likes, comments, retweets, upvotes) — they do not track price,
market capitalization, or trading volume. If you're looking for price data,
that would be a separate future project (see the "What this doesn't cover"
section below).

---

## Accessing the Dashboards

**Power BI Desktop (how this project is currently shared)**

1. Get the `.pbix` file from the `reports/` folder:
   - `NoodlesCrypto_TopPerformers.pbix`
   - `NoodlesCrypto_ExecutiveDashboard.pbix`
2. Open it with Power BI Desktop (free to install)
3. If prompted for a data connection and you have access to the underlying
   MySQL database, click **Refresh** to load the latest data. If not, the
   report will still open and show the data as of the last save.

These reports were not published to Power BI Service for this project, so
there is no web link to sign into — see the Technical Runbook if that gets
set up later.

---

## Available Dashboards

### 1. Top Performers (`NoodlesCrypto_TopPerformers.pbix`)

**Purpose:** Identify which currencies get the most social media buzz, and
compare Twitter vs Reddit.

**Pages:**
- **Executive Dashboard** — a table of all 33 active currencies ranked by
  total engagement, a Top 10 bar chart, and 3 KPI cards (Total Engagements,
  Average Engagement Score, distinct currency count)
- **Time Series Analysis** — a line chart of engagement over time, an area
  chart of post volume, and a stacked column chart breaking down likes,
  comments, and retweets by date
- **Platform Analysis** — a table and two bar charts comparing Twitter and
  Reddit head-to-head
- **Currency Deep Dive** — pick one currency from the slicer and see its
  engagement trend and platform split

**How to use:**
- On the Executive Dashboard, the bar chart is fixed to the Top 10 by
  engagement
- On Currency Deep Dive, click a currency symbol in the left-hand list to
  filter the trend chart and donut chart to just that currency
- Right-click a row on the Executive Dashboard table → **Drill through →
  Currency Deep Dive** to jump straight to that currency's detail page

**Known limitation:** on Currency Deep Dive, clicking a currency updates the
donut chart correctly but does not filter the line chart — the line chart
shows the overall trend across all currencies regardless of selection. This
is because the underlying data view for that chart doesn't have a
currency-level breakdown (see the Technical Runbook for the full
explanation if you're curious).

### 2. Executive Dashboard (`NoodlesCrypto_ExecutiveDashboard.pbix`)

**Purpose:** A more advanced, executive-level view with day-over-day growth
tracking and a fixed trend chart (see below).

**Pages:**
- **Executive Overview** — 4 KPI cards, an engagement trend line chart, a
  ranked table of top tokens, a platform-share donut chart, and a platform
  quality column chart
- **Platform Performance Analysis** — a corrected trend chart comparing
  Twitter and Reddit day-by-day (with a date slicer to zoom into specific
  windows), plus a platform snapshot table
- **Token Drill-through** — same idea as Currency Deep Dive above, reached
  by right-clicking a token on the Executive Overview table

**How to use:**
- On Platform Performance Analysis, use the date range slicer (top of page)
  to zoom into a specific window — this is especially useful since Twitter's
  sample data only spans about a month (Oct 21 – Nov 18, 2025) while
  Reddit's spans several years
- Right-click any token on the Executive Overview table → **Drill through →
  Token Drill-through**

---

## Common Tasks

### Filter by Date Range
1. Locate the date slicer (Platform Performance Analysis page)
2. Type or pick a start and end date in the two date boxes, or drag the
   slider handles
3. All visuals on that page update automatically

### Export Data
1. Click the **"..."** menu in the top corner of any visual
2. Select **Export data**
3. Choose `.xlsx` or `.csv`
4. Open in Excel for further analysis

### Drill Through to a Specific Currency
1. On the Executive Dashboard/Overview page, find the currency in the table
2. Right-click that row
3. Select **Drill through → Currency Deep Dive** (or **Token Drill-through**)
4. Use the back arrow (top-left of the page) to return

---

## Understanding Key Metrics

| Metric | What it means |
|---|---|
| TotalEngagements | Total interaction count for a currency (likes + weighted retweets + weighted comments, summed across all its posts) |
| AvgEngagementScore | The average "quality" of engagement per post — a custom weighted score, not a percentage |
| TotalLikes / TotalComments / TotalRetweets | Raw counts of each interaction type |
| ActiveCurrencies / Active Tokens | How many of the 78 tracked currencies actually had at least one social post (33) |
| Token Engagement Rank | Where a currency ranks against all others by total engagement (1 = highest) |
| Engagement Growth % | Day-over-day percent change in engagement (Executive Dashboard file only) |
| Engagements 7D Avg | 7-day rolling average of engagement (Executive Dashboard file only) |

---

## What This Doesn't Cover

To set expectations clearly:

- **No price or market cap data.** The underlying dataset includes a
  currency market snapshot file (`v2_token_overview.json`), but it was never
  joined against the engagement data — this project answers "how much buzz"
  a currency gets, not "did the price move."
- **No prediction or correlation claims.** The dashboards report what
  happened (engagement volume and trend), not why it happened or whether it
  predicts anything about price.
- **Sample data, not live data.** The Twitter and Reddit files are a
  provided sample for this internship project, not a live, continuously
  updating feed.

---

## Tips & Best Practices

✅ **DO**
- Use the date slicer to compare specific time windows
- Right-click a currency to drill into its details
- Export data to Excel if you need it for offline analysis

❌ **DON'T**
- Treat AvgEngagementScore as a percentage — it's a weighted point score,
  not bounded 0-100
- Expect the Currency Deep Dive line chart to filter by currency (see Known
  Limitation above)
- Use this dashboard to make price or trading decisions — it does not
  contain price data

---

## FAQ

**Q: How often is the data updated?**
A: Manually, by opening Power BI Desktop and clicking Refresh. There is no
automatic scheduled refresh set up for this project.

**Q: Why does the Currency Deep Dive line chart not change when I pick a
different currency?**
A: This is a known, documented limitation — see above. The donut chart and
table on that page do filter correctly.

**Q: Can I add my own visuals?**
A: Yes, if you have Power BI Desktop and the `.pbix` file — all the
underlying measures and relationships are already built and reusable.

**Q: Who do I contact with questions?**
A: Pavendhar Tamilarasu (project author) — see Technical Runbook for
project context.
