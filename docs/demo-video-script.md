# Demo Video Script (Short Version) — Noodles Crypto Analytics
**Target length:** ~5-6 minutes
**Author:** Pavendhar Tamilarasu

---

## 1. Intro (10 sec)

**DO:** Title slide of presentation.

**SAY:**
> "Hi, I'm Pavendhar. This is a project from my data analytics internship —
> a pipeline that measures social media engagement for 78 cryptocurrencies
> across Twitter and Reddit, from raw JSON files to interactive Power BI
> dashboards."

---

## 2. Jupyter (2 min)

**DO:** Open `05_data_warehouse_design.ipynb`, scroll to the `normalize_symbol()` cell.

**SAY:**
> "Twitter data has messy symbols like '$DAIUSD' — this function cleans
> them so tweets join correctly to the right currency. After joining,
> 1,144 tweets and 1,538 Reddit posts matched — 2,682 total engagement
> records."

**DO:** Switch to `06_powerbi_prep.ipynb`, show Cell 9's output.

**SAY:**
> "This checks for broken links, missing data, and duplicates. It came
> back 100% — all 2,682 records passed, zero bad rows."

**DO (optional):** Point at the "Twitter likes total" line in Cell 8's output.

**SAY:**
> "I also specifically verified Twitter's likes weren't zero — a common
> bug in this project if the data isn't unpacked correctly. Mine came back
> healthy at 7,284."

---

## 3. Power BI (2-3 min)

**DO:** Open `NoodlesCrypto_TopPerformers.pbix`, Executive Dashboard page. Click the TotalEngagements column header to sort.

**SAY:**
> "33 currencies actually have engagement out of 78 tracked. This ranks
> them."

**DO:** Click a currency in a slicer (or right-click a table row → Drill through).

**SAY:**
> "Clicking a currency filters everything else on the page — that's
> cross-filtering, from the relationships I built between the tables."

**DO:** Switch to `NoodlesCrypto_ExecutiveDashboard.pbix`, Platform Performance Analysis page. Point at the two colored lines.

**SAY:**
> "This chart was originally broken — it collapsed into one flat point
> because it used the wrong date field. I rebuilt it on the correct table,
> and now it shows a real trend for both platforms. Reddit has more posts,
> but Twitter's engagement score per post is over double Reddit's."

---

## 4. Wrap-up (15 sec)

**DO:** Thank-you slide.

**SAY:**
> "Python for ETL, MySQL for storage, Power BI with 19 DAX measures for
> reporting. Thanks for watching."

---

## Quick prep checklist before recording

- [ ] Both notebooks open in Jupyter, scrolled to the right cells
- [ ] Both `.pbix` files open in Power BI Desktop
- [ ] Presentation open on the title slide
- [ ] Loom (or chosen tool) set to "Screen" or "Screen + Cam"
