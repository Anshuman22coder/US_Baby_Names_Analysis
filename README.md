# 👶 US Baby Names Trend & Regional Popularity Analysis (1980–2010)

An end-to-end business intelligence and data analytics project analyzing three decades of US birth records. This project focuses on decade-level popularity transitions, regional clustering, and gender volume distribution using **SQL Server (T-SQL)**, **Power BI Desktop**, and **advanced DAX**.

---

## 📌 Project Overview
* **Domain:** Demographics & Societal Trend Analytics
* **Time Span:** 1980 – 2010 (Bucketed into 1980s, 1990s, and 2000s)
* **Scale:** ~2 Million birth entries tracked across all 50 US states and 4 distinct geographical regions
* **Core Technologies:** 
  * **SQL Server (T-SQL):** Common Table Expressions (CTEs), Window Functions (`DENSE_RANK() OVER (PARTITION BY ...)`), and conditional aggregation.
  * **Power BI Desktop:** Multi-page interactive dashboard, dynamic custom tooltips, matrix hierarchies.
  * **DAX:** Iterator functions (`MAXX`), dynamic context ranking (`RANKX`), and visual scope evaluation (`ISINSCOPE`).

---

## 📊 High-Level KPI Summary
| Metric | Value | Analytical Interpretation |
| :--- | :--- | :--- |
| **Total Recorded Volume** | **~2,000,000** | Total birth sample evaluated (1980–2010) |
| **Male Birth Records** | **967K** | ~48.4% of total recorded births |
| **Female Birth Records** | **1,000,000** | ~50.0% of total recorded births |
| **Male / Female Ratio** | **1.29** | Ratio benchmark across primary name frequencies |
| **Reigning Decade Winner (80s & 90s)** | **Michael** | #1 national name across two consecutive decades (0.67M in the 80s, 0.46M in the 90s) |
| **New Millennium Successor (2000s)** | **Jacob** | Surpassed Michael starting in 1999–2000 |

---

## 🖥️ Power BI Interactive Dashboard

### Page 1: Longitudinal & Decade Name Trends
*Tracks national name dominance, multi-decade decay curves, and state-level geographic concentrations.*

![Page 1 - US Baby Names Trends](<Screenshots/Home.png>)

* **Decade Summary:** Highlights the dominance of *Michael* through the 1980s and 1990s, followed by the transition to *Jacob* in the 2000s.
* **Decay & Diversification Line Chart:** Demonstrates how peak name concentration dropped sharply over time (the top name in 1980 commanded >68K births, falling below 25K by 2010 due to naming diversification).
* **State Volume Distribution:** Breakdown of leading baby name counts across California, New York, Texas, Pennsylvania, and Illinois.

---

### Page 2: Regional Clustering & Gender Matrix
*Hierarchical regional matrix analysis and year-over-year gender volume breakdown.*

![Page 2 - Regional & Gender Distribution](<Screenshots/Second Page.png>)

* **Regional Volume Share:** Evaluates birth concentrations across the **South (1.03M)**, **Midwest (768K)**, **Northeast (644K)**, and **West (606K)**.
* **Hierarchical Matrix with DAX:** Dynamic Top-N ranking per region/decade using `RANKX` with leaf-level evaluation.
* **Longitudinal Cohort Matrix:** Tracks individual naming trajectories (*Michael*, *Christopher*, *Matthew*, *Joshua*, *Jessica*) across each individual year from 1980 to 2010.

---

## 🧠 Key Insights & Business Findings

1. **The Fall of the Monolith Name:**
   * In 1980, the top boy name (*Michael*) accounted for **~68,000+** births alone. By 2010, the #1 name (*Jacob*) peaked at only **~22,000**, revealing strong long-term cultural diversification and name fragmentation in the United States.
2. **Regional Popularity Divergence:**
   * While *Michael* was ubiquitous across all regions in the 1980s, southern states favored *Christopher* and *James* in high numbers earlier than northern and midwestern counterparts.
3. **Female Trend Volatility vs. Male Stability:**
   * Male names exhibited multi-decade stability (*Michael* held rank 1 for two full decades). Female names (*Jennifer*, *Amanda*, *Jessica*, *Emily*) surged rapidly but also saw steeper, shorter decade life-cycles.

---

## 🛠️ Technical Implementation: SQL vs. DAX

### 1. Replicating SQL Window Functions in DAX (`DENSE_RANK`)

#### SQL Approach:
```sql
WITH regional_totals AS (
    SELECT region, decade, name, SUM(births) AS total_births 
    FROM state_to_region
    WHERE region IS NOT NULL
    GROUP BY region, decade, name
)
SELECT *,
    DENSE_RANK() OVER (
        PARTITION BY region, decade 
        ORDER BY total_births DESC
    ) AS rnk 
FROM regional_totals;
```

#### Power BI DAX Measure Equivalent:
```dax
Regional_Name_Rank = 
IF(
    ISINSCOPE(names[name]),
    RANKX(
        ALLSELECTED(names[name]),
        [Total_Births],
        ,
        DESC,
        Dense
    )
)
```
* **Deduction:** The visual row context (`Decade` $\rightarrow$ `Region`) handles the partition implicitly. `ALLSELECTED(names[name])` clears the row-level filter on the current name to evaluate all competitor names inside that region/decade slice, while `ISINSCOPE` prevents rank calculations on subtotal headers.

---

### 2. Dynamic Peak Birth Calculation per Year

#### SQL Grouping:
```sql
SELECT year, MAX(tot_births) AS peak_births
FROM (
    SELECT year, name, SUM(births) AS tot_births
    FROM names
    GROUP BY year, name
) sub
GROUP BY year;
```

#### Power BI DAX Measure:
```dax
Top_Births_Per_Year = 
MAXX(
    VALUES(names[name]),
    CALCULATE(SUM(names[births]))
)
```
* **Context Transition:** `VALUES()` creates an in-memory unique list of names for the axis year. `CALCULATE()` converts the current iteration name into an active filter context, computing the exact sum before `MAXX()` extracts the single highest count.

---

## 📂 Repository Structure
```text
us-baby-names-analytics/
│
├── README.md
├── pbix/
│   └── Us_baby_report.pbix         <-- Interactive Power BI report file
├── SQL/
│   ├── 01_decade_kpis.sql          <-- SQL window ranking queries
│   └── 02_regional_analysis.sql    <-- State-to-region mapping CTEs
├── images/
│   ├── dashboard_page1.png         <-- Screenshot: Trend Overview
│   └── dashboard_page2.png         <-- Screenshot: Regional Matrix
```

---

## 🚀 How to Run Locally

1. Clone the repository:
```bash
git clone [https://github.com/Anshuman22coder/US_Baby_Names_Analysis.git](https://github.com/Anshuman22coder/US_Baby_Names_Analysis.git)
```
2. Open the SQL scripts in **SSMS** or any SQL IDE to inspect query logic.
3. Open `Us_baby_report.pbix` in **Power BI Desktop** to explore the data models, DAX measures, and drill-through visual layers.