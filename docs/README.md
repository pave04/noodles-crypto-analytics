# Noodles Crypto Analytics Platform

![Executive Dashboard](docs/screenshots/executive-dashboard.png)

## Project Overview

A social-engagement analytics platform for 78 cryptocurrencies, built end to
end during a data analytics internship — Python ETL, a MySQL star-schema
data warehouse, and interactive Power BI dashboards.

**Scope note:** this project measures social media engagement (likes,
comments, retweets, upvotes) on Twitter and Reddit — it does not track
price, market capitalization, or trading volume. See the User Guide for
details.

**Key Features:**
- Python ETL pipeline (pandas + SQLAlchemy) with symbol normalization and
  public_metrics flattening
- Star schema data warehouse in MySQL (3 dimensions, 1 fact table)
- 4 pre-aggregated SQL views + 19 DAX measures powering 2 Power BI reports
- 100% data quality score across all 2,682 loaded engagement records
- Drill-through, cross-filtering, and date-range slicers across 7 report pages

## Architecture

```
JSON Files → Python ETL (pandas) → MySQL (noodles_dw, star schema) → Power BI Dashboards
```

Full diagram: [docs/architecture-diagram.png](docs/architecture-diagram.png)

## Technology Stack

| Layer | Technology |
|---|---|
| ETL | Python 3.9+, pandas, SQLAlchemy, PyMySQL |
| Data Warehouse | MySQL 8.x |
| Visualization | Power BI Desktop, DAX, ODBC |
| Orchestration | Jupyter Notebook |

## Project Structure

```
noodles-analytics/
├── 05_data_warehouse_design.ipynb   (Task 5 — star schema ETL)
├── 06_powerbi_prep.ipynb            (Task 6 — aggregations + Power BI views)
├── reports/
│   ├── NoodlesCrypto_TopPerformers.pbix
│   └── NoodlesCrypto_ExecutiveDashboard.pbix
├── docs/
│   ├── architecture-diagram.png
│   ├── data-dictionary.xlsx
│   ├── technical-runbook.md
│   ├── user-guide.md
│   ├── demo-presentation.pptx
│   ├── demo-video-script.md
│   ├── final-checklist.md
│   └── screenshots/
└── README.md
```

## Data Model

**Dimensions:** DimCurrency (78 rows), DimDate (1,300+ rows), DimPlatform (2 rows)
**Fact:** FactSocialEngagement (2,682 rows — 1,144 Twitter + 1,538 Reddit)
**Grain:** one row per social media post

Full data dictionary: [docs/data-dictionary.xlsx](docs/data-dictionary.xlsx)

## Key Insights

- **Reddit drives volume, Twitter drives quality per post** — Reddit logged
  1,538 posts vs Twitter's 1,144, but Twitter's average engagement score
  (700.14) is more than double Reddit's (320.28)
- **Engagement is concentrated** — Magic Internet Money (MIMUSD) alone
  accounts for 288 of 2,682 total engagements, roughly 1 in 9
- **Only 33 of 78 tracked currencies (42%) have any social presence** at all
- **Platform sample windows differ** — Reddit spans 2020–2025 (617 distinct
  days), Twitter spans Oct–Nov 2025 only (11 distinct days) — any platform
  comparison needs a matched date range, which the dashboards support via a
  date slicer

## Achievements

- Processed 2,682 real engagement records across two independent social
  platforms into a single, queryable warehouse
- Built and debugged a working star schema with verified referential
  integrity (0 orphaned records)
- Diagnosed and fixed a real Power BI bug — a platform trend chart that
  collapsed to one data point — by rebuilding it against the correct
  pre-aggregated view
- Delivered 2 Power BI reports, 7 pages, 19 DAX measures, drill-through, and
  cross-filtering
- Documented the full pipeline for handoff: architecture diagram, data
  dictionary, technical runbook, and user guide

## Documentation

- Technical Runbook → [docs/technical-runbook.md](docs/technical-runbook.md)
- User Guide → [docs/user-guide.md](docs/user-guide.md)
- Architecture Diagram → [docs/architecture-diagram.png](docs/architecture-diagram.png)
- Data Dictionary → [docs/data-dictionary.xlsx](docs/data-dictionary.xlsx)
- Presentation Slides → [docs/demo-presentation.pptx](docs/demo-presentation.pptx)
- Final Checklist → [docs/final-checklist.md](docs/final-checklist.md)

## Demo

Demo video: https://www.loom.com/share/1e1ad6f3c8a744db9878cd4c0f021b48
Presentation: [docs/demo-presentation.pptx](docs/demo-presentation.pptx)

## What This Doesn't Cover

To be upfront about scope: this project does not analyze price or market
capitalization, and does not test whether social engagement predicts price
movement. The source data includes a market snapshot file that was
deliberately left out of the warehouse — a natural next step would be
joining it in to test that question, but it wasn't part of this build.

## Contact

Pavendhar Tamilarasu
