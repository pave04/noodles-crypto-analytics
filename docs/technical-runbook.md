# Technical Runbook: Noodles Crypto Analytics
**Author:** Pavendhar Tamilarasu

## System Overview

A Python-based ETL pipeline that loads currency, Twitter, and Reddit engagement
data from JSON files into a MySQL star-schema data warehouse (`noodles_dw`),
with Power BI dashboards connected via ODBC for reporting.

Everything runs from two Jupyter notebooks — there are no standalone `.py`
script files in this project. All ETL logic, aggregation, and view creation
lives in:

- `05_data_warehouse_design.ipynb` — loads the star schema (dimensions + fact)
- `06_powerbi_prep.ipynb` — builds aggregation tables, calculated columns, and
  the 4 Power BI views

---

## Daily Operations

### 1. Manual ETL Execution

```bash
# Activate Python environment
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies (once)
pip install pandas sqlalchemy pymysql python-dotenv matplotlib seaborn openpyxl

# Open Jupyter in the project folder
jupyter notebook
```

Then, in order:

1. Open `05_data_warehouse_design.ipynb`
2. Run Cell 1 (MySQL connection) through Cell 15 (schema export + close
   connection), top to bottom
3. Open `06_powerbi_prep.ipynb`
4. Run in this exact order (not top-to-bottom by cell number — some cells
   only *define* functions and a later cell *runs* them):
   Cell 1 → Cell 2 → Cell 5 (aggregation functions) → Cell 5b (run
   aggregations) → Cell 3 (calculated column functions) → Cell 4 (run
   calculated columns) → Cell 6 (create views) → Cell 7 (validate) → Cell 8
   (data quality + charts) → Cell 9 (optional reject export)

**Expected duration:** under 1 minute for the full pipeline on this dataset
(2,682 fact rows) — this is not a 5-10 minute job at this data volume,
unlike a production-scale pipeline.

**Success indicators:**
- Every cell prints a line starting with `✓`
- Cell 7 shows all 4 views with row counts greater than 0
  (`vw_ExecutiveDashboard`: 33, `vw_TimeSeries`: 619, `vw_SocialAnalytics`:
  41, `vw_PlatformDaily`: 628)
- Cell 9's data quality score prints 100.0%, or close to it

### 2. Automated Execution (optional — not set up in this project)

This project runs manually, cell by cell, in Jupyter. It was not scheduled
via cron or Windows Task Scheduler. If this were productionized, the
equivalent commands would be:

**Linux/Mac (cron):**
```bash
# Run daily at 6 AM
0 6 * * * cd /path/to/noodles-analytics && jupyter nbconvert --execute 06_powerbi_prep.ipynb
```

**Windows (Task Scheduler):**
- Create Basic Task → Trigger: Daily at 6:00 AM
- Action: Start `python.exe`, arguments to execute the notebook via
  `nbconvert`, working directory set to the project folder

### 3. Power BI Refresh

**Manual refresh (what this project actually uses):**
1. Open the `.pbix` file in Power BI Desktop
2. Home → Refresh
3. Wait a few seconds for completion (dataset is small — well under the
   30-60 second range a larger warehouse might take)

**Scheduled refresh (Power BI Service):** not configured for this project —
publishing to the Service was optional (Task 8 §10) and was not completed.
See the Power BI Service section of Task 8 if this is set up later.

---

## Monitoring & Validation

### Check Data Quality

The validation step for this project lives inside `06_powerbi_prep.ipynb`,
not a separate log file:

- **Cell 8** — checks raw-vs-aggregated currency counts match (33 = 33),
  checks for nulls in `CurrencySummary_Enriched`, charts the engagement
  category distribution, prints the top 10 currencies by engagement
- **Cell 9** (optional) — scans `FactSocialEngagement` for three specific
  problems and exports any bad rows to `data/rejects/bad_records_<timestamp>.csv`:
  1. Orphaned foreign keys (a fact row pointing to a currency that doesn't exist)
  2. Null engagement metrics (`Likes` or `EngagementScore` is NULL)
  3. Duplicate `PostId` values on the same platform

On the last run, this returned a **100.0% data quality score (2,682/2,682
good)** with zero bad records.

### Run the SQL QA Queries (Task 8 §1.4)

Before trusting the Power BI trend charts, run these two queries in MySQL
Workbench:

```sql
USE noodles_dw;

-- A) Twitter must have likes and non-zero avg score
SELECT dp.PlatformName,
       COUNT(*) AS fact_rows,
       SUM(f.Likes) AS total_likes,
       ROUND(AVG(f.EngagementScore), 4) AS avg_engagement_score,
       COUNT(DISTINCT dd.FullDate) AS distinct_post_days
FROM FactSocialEngagement f
JOIN DimPlatform dp ON f.PlatformKey = dp.PlatformKey
JOIN DimDate dd ON f.DateKey = dd.DateKey
GROUP BY dp.PlatformName;

-- B) Platform daily view must have multiple dates per platform
SELECT PlatformName,
       COUNT(DISTINCT FullDate) AS distinct_days,
       MIN(FullDate) AS min_date,
       MAX(FullDate) AS max_date
FROM vw_PlatformDaily
GROUP BY PlatformName;
```

**Pass criteria confirmed on this project:** Twitter `total_likes = 7,284`,
`avg_engagement_score = 18.89`, `distinct_post_days = 11`; Reddit
`distinct_days = 617` (spanning 2020-12-16 to 2025-11-16), Twitter
`distinct_days = 11` (spanning 2025-10-21 to 2025-11-18).

---

## Troubleshooting

### Issue: Python script fails with ImportError
**Symptom:** `ModuleNotFoundError: No module named 'pandas'`

**Solution:**
```bash
pip install pandas sqlalchemy pymysql python-dotenv matplotlib seaborn openpyxl
```

### Issue: Database connection fails with "Access denied ... using password: NO"
**Symptom:** `pymysql.err.OperationalError: (1045, "Access denied for user 'root'@'localhost' (using password: NO)")`

**Root cause encountered on this project:** connection string built without
a password even though root had one set locally.

**Solution:** include the real password in the connection string, and
URL-encode any special characters (e.g. `@` becomes `%40`, or use
`urllib.parse.quote_plus()` to do this automatically):
```python
from sqlalchemy import create_engine
from urllib.parse import quote_plus

password = quote_plus("your_real_password")
engine = create_engine(f"mysql+pymysql://root:{password}@localhost:3306/noodles_dw")
engine.connect()
print("Connection successful!")
```

### Issue: Aggregation tables empty or `InfluenceLevel` shows all nulls
**Symptom:** `CurrencySummary_Enriched.InfluenceLevel` is `None` for every row.

**Root cause encountered on this project:** the original `pd.cut()` bins for
`InfluenceLevel` assumed `AvgEngagementScore` is normalized to a 0-1 scale.
This project's real scores range from roughly 1 to 700+, so every value
fell outside the bin edges and returned null.

**Status:** left as-is for this submission (matches the task's original
formula) rather than silently patched — documented here and in the Task 6
DAX/aggregation notes so a future maintainer knows the bins need rescaling
to the real data range (e.g. `[0, 15, 35, inf]`) before this column is
usable.

### Issue: Power BI Navigator shows no views, or only dBASE/Excel/Access drivers
**Symptom:** ODBC connector dropdown in Power BI only lists built-in Windows
data sources.

**Solution:**
1. Confirm the MySQL ODBC driver is actually installed (64-bit) —
   download from https://dev.mysql.com/downloads/connector/odbc/
2. Confirm the DSN was created in the **64-bit** ODBC Data Source
   Administrator (`C:\Windows\System32\odbcad32.exe`, not the SysWOW64
   32-bit version)
3. Confirm the DSN's Database field points to `noodles_dw`, not the
   default `mysql` system schema
4. Re-run Task 6 §15b's `SHOW FULL TABLES WHERE Table_type = 'VIEW'` in
   MySQL Workbench to confirm the 4 views genuinely exist server-side

### Issue: `Token Engagement Rank` DAX measure shows 1 for every row
**Symptom:** `RANKX` returns rank 1 for every currency in a table visual,
instead of counting up.

**Root cause encountered on this project:** the original formula ranked
using the `[Total Engagements]` measure directly inside `RANKX`, which lost
per-row context in this table.

**Solution used:**
```dax
Token Engagement Rank =
RANKX(
    ALL(vw_ExecutiveDashboard),
    CALCULATE(SUM(vw_ExecutiveDashboard[TotalEngagements])),
    ,
    DESC,
    Dense
)
```
Wrapping the ranking value in `CALCULATE(SUM(...))` against
`ALL(vw_ExecutiveDashboard)` restores each row's own context before ranking.

### Issue: Power BI trend chart on Platform Performance page shows only one day
**Symptom:** X-axis collapses to a single day-of-month value (e.g. "27")
instead of a real date range.

**Root cause:** built on the wrong table/axis — either `vw_SocialAnalytics`
(which has no post date at all) or a `Day`-only field instead of the full
`FullDate` column.

**Solution:** rebuild the visual using `vw_PlatformDaily[FullDate]`
specifically on the X-axis (removing any auto-added `Day` hierarchy level),
with `PlatformName` on Legend. Confirmed working on this project — the
resulting chart shows two distinct lines (Twitter, Reddit) across real
dates.

---

## File Locations

| Resource | Path |
|---|---|
| JSON source files | project root (uploaded per task, e.g. `v2_token.json`, `v4_x_tweets.json`) |
| Notebooks | `05_data_warehouse_design.ipynb`, `06_powerbi_prep.ipynb` in project root |
| Power BI files | `reports/NoodlesCrypto_TopPerformers.pbix`, `reports/NoodlesCrypto_ExecutiveDashboard.pbix` |
| Documentation | `docs/` |
| Reject/QA exports | `data/rejects/` (created by Task 6 Cell 9) |
| Power BI data summary CSV | `reports/powerbi_data_summary.csv` |

---

## Database Information

- **Server:** localhost
- **Database:** `noodles_dw`
- **Port:** 3306
- **Authentication:** MySQL `root` user, password set locally (not committed
  to any file — entered directly in notebook connection cells and Power BI's
  ODBC DSN)
- **ODBC DSN name used:** `NoodlesDW`

---

## Key Contacts

- **Data Engineer:** Pavendhar Tamilarasu
- **Program:** MVP Studio internship — Data & BI track
- **Mentor support:** MVP Studio Project Portal / Question Hub (per the
  internship handbook)

---

## Backup & Recovery

**Database backup** — not performed for this project (a personal/internship
learning project on local MySQL, not a production system). For a real
deployment, a `mysqldump` export would replace this section:
```bash
mysqldump -u root -p noodles_dw > noodles_dw_backup.sql
```

**Code backup** — notebooks and Power BI files are kept locally; this
project was not pushed to a Git repository during development. See §7 of
Task 9 for the README that would accompany a GitHub push.
