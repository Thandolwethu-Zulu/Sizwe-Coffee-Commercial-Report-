# 📊 Data Analytics Portfolio

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-yellow)
![DAX](https://img.shields.io/badge/DAX-Time%20Intelligence%20%26%20Iterators-blue)
![Power Query](https://img.shields.io/badge/Power%20Query-Data%20Prep-green)
![Data Analytics](https://img.shields.io/badge/Data%20Analytics-Portfolio-orange)

Project Workflow: Raw CSV Data ➡️ Power Query Cleaning & Locale Remediation ➡️ Star Schema & Fiscal Date Model ➡️ DAX Measure Engineering ➡️ 4-Page Power BI Dashboard & Executive Memo

---
## 🧠 Project Name: Sizwe Coffee — Commercial & Receivables Analytics

---
## 🗺️ Overview

As a Data Analyst supporting Sizwe Coffee, I was tasked by **Nomsa Dlamini** (Financial Director) and **Bongani Twala** (National Sales Manager) to replace a manual, two-day monthly Excel reporting process with a single, trusted **Power BI** reporting solution[cite: 2]. The objective was to build an automated star-schema data model and interactive 4-page dashboard covering **FY2024 – FY2025 (1 July 2023 – 30 June 2025)** so leadership can select any period and immediately see how the business is trading and where cash is tied up[cite: 2, 4].

---

## 🎯 The Problem

Sizwe Coffee roasts and distributes coffee to Wholesale and Subscription accounts across South Africa[cite: 2]. Previously, someone exported sales and debtors data into Excel every month, manually rebuilt pivot tables, and emailed a workbook that took two days to produce and lacked stakeholder trust[cite: 2]. To replace the Excel pack and diagnose working capital performance, the model and report needed to answer four core executive questions[cite: 2, 4, 5]:

1. **How are we trading?** Are revenue, gross margin, and year-over-year growth moving in the right direction across Sizwe's July–June financial year[cite: 2, 4]?
2. **What are we selling, and to whom?** How has product line mix shifted between FY2024 and FY2025, and how does mix impact gross margin across channels[cite: 2, 4]?
3. **Who is selling it?** How are sales reps and territories tracking against monthly revenue targets, and does rep visit activity translate into revenue[cite: 2, 4]?
4. **Where is our cash?** Why has customer payment turnaround slowed across the two years, which metric accurately reflects working capital risk, and which accounts require immediate collections follow-up[cite: 4, 5]?

#### 📌 Core Analytical Hypotheses & Reporting Guardrails Tested

- **Hypothesis 1 (Receivables Velocity Shift):** Customer settlement times deteriorated materially at a specific inflection point during the 24-month window rather than fluctuating randomly[cite: 5].
- **Hypothesis 2 (Root Cause — Terms vs. Delinquency):** The slowdown in collections can be traced directly in the receivables ledger to either a deliberate extension of Sizwe's contractual payment terms or genuine customer delinquency past unchanged due dates[cite: 5].
- **Hypothesis 3 (Metric Distortion — Weighted vs. Unweighted):** High-volume, low-value Subscription accounts pay faster than large Wholesale accounts, causing a simple unweighted average (`Avg Days to Settle`) to understate true Rand working capital lock-up compared to `Weighted Days to Settle`[cite: 2, 4, 5].

Before analyzing trends, two structural modeling rules had to be enforced: aligning every time-intelligence calculation to Sizwe's **July 1 – June 30 financial year** (rather than calendar years)[cite: 2, 3], and accounting for **right-censored invoices** in the final months of FY2025 (April–June 2025) where unpaid open invoices distort average settlement duration downward if viewed without open-balance context[cite: 4, 5, 26].

----

## 💻 Tools & Skills Used

**Tools**
- Power BI Desktop (Import Mode)[cite: 5]
- Power Query (M)
- DAX (Data Analysis Expressions)[cite: 1]
- Microsoft Word / PDF Documentation[cite: 5]
- Git and GitHub

**Skills**
- Star Schema Data Modeling & Conformed Dimensions[cite: 2]
- DAX — Fiscal Time Intelligence (`DATESYTD`, `SAMEPERIODLASTYEAR`), `SUMX` Iterators, Value-Weighted Averages, Dynamic As-At Aging Buckets[cite: 3, 4]
- Control Total Reconciliation (`R921,000,020.82` across independent fact tables)[cite: 2, 3]
- Data Quality Auditing & Locale Remediation
- Executive Dashboard Design & Visual Storytelling[cite: 1, 4]
- Executive Financial Memo & Architecture Documentation[cite: 5]

## 📦 The Dataset

The full dataset consists of eight CSV files covering two full financial years (**1 July 2023 to 30 June 2025**) across Wholesale and Subscription accounts nationwide[cite: 2].

| File | Rows | Description |
| :--- | :--- | :--- |
| `sizwe_customers.csv` | 1,400 | Trading accounts with channel, segment, province, credit limit, and payment terms[cite: 2, 6, 7] |
| `sizwe_products.csv` | 23 | Coffee SKU master catalogue across product lines and pack sizes[cite: 2, 8] |
| `sizwe_reps.csv` | 18 | Sales representatives, territories, hire dates, and monthly revenue targets[cite: 2, 10, 11] |
| `sizwe_sales_orders.csv` | 48,144 | Order headers linking customers, reps, order dates, and sales channels[cite: 2, 12, 13] |
| `sizwe_sales_lines.csv` | 168,172 | Order line-item grain capturing units sold, unit price, discounts, revenue, and COGS[cite: 2, 14, 15, 16] |
| `sizwe_ar_ledger.csv` | 48,144 | Customer invoices with due dates, settlement dates, and settled amounts[cite: 2, 17, 18] |
| `sizwe_rep_visits.csv` | 9,400 | Recorded field sales visits by representative, customer, and date[cite: 2] |
| `sizwe_trade_promotions.csv` | 640 | Historical customer promotional windows (evaluated and excluded from active star schema)[cite: 2] |

----

## 🗂️ Schema Architecture

The semantic model is structured as a multi-fact star schema using single-direction, one-to-many ($1 \to \ast$) relationships from conformed dimension tables (`DimDate`, `Customers`, `Products`, `Reps`) to transactional fact tables (`Sales_lines`, `Sales_orders`, `Ar_ledger`, `Rep_visits`)[cite: 2, 3]:

- **Resolving the Invoice-to-Order Disconnect:** `Sales_orders` (`order_id`) and `Ar_ledger` (`invoice_no`) do not share a foreign key[cite: 2]. Instead of forcing a bidirectional or many-to-many bridge that duplicates rows, both fact tables connect independently to `DimDate` and `Customers`[cite: 2, 3]. This preserves exact cross-filtering by customer, channel, and fiscal period while locking both control totals at **`R921,000,020.82`**[cite: 2, 3].
- **Excluded Table:** `Trade_promotions` (640 rows) was omitted from the active schema to prevent many-to-many date-range ambiguity across order lines[cite: 2].

*(Tip: Take a screenshot of your Power BI Model View diagram and paste it here on GitHub!)*

----

## ⚠️ Data Quality Issues & Remediation

All data auditing and preparation were completed in Power Query prior to loading into the model[cite: 3]:

1. **Control Total Reconciliation (`R921,000,020.82`):** Validated that total line-item sales revenue (`Sales_lines[line_revenue]`) and total invoiced value (`Ar_ledger[invoice_amount]`) reconcile to the exact cent with zero relationship fan-out[cite: 2, 3].
2. **Alphanumeric ID Misclassification:** Hyphenated representative IDs (`R-01`, `R-15`) were initially misread by Power Query's automatic type detection as negative currency values (`-1.00`, `-15.00`) or conversion errors in `Customers`, `Reps`, `Sales_orders`, and `Rep_visits`[cite: 7, 10, 13]. Remediated by stripping auto-typed steps and explicitly casting all primary and foreign keys as Text[cite: 3].
3. **Regional Decimal Parsing Errors:** Period-delimited monetary and percentage fields (`list_price`, `unit_cost`, `line_revenue`, `cogs`, `invoice_amount`, `amount_settled`) threw type errors under South African regional comma settings[cite: 9, 15, 16, 18]. Fixed using locale-specific parsing (`English (United States)`) into Fixed Decimal currency types[cite: 3].
4. **Model Hygiene:** Hidden all surrogate and foreign keys (`customer_id`, `order_id`, `invoice_no`, `rep_id`, `sku`) and disabled default auto-summarization on non-additive fields such as `credit_limit`, `payment_terms`, `unit_price`, and `Year`[cite: 3].

## 📊 Dashboards

The report is organized into four stakeholder-focused pages designed to fit a laptop screen without scrolling[cite: 4, 5]:

- **Page 1 — Trading (`How are we trading?`):** Executive KPI cards (`Revenue`, `Revenue YoY`, `Gross Margin`, `Gross Margin Change (pp)`), monthly revenue and margin trend across FY2024–FY2025, and channel performance split[cite: 3, 4, 21].
- **Page 2 — Product & Mix (`What are we selling, and to whom?`):** Product line revenue share, FY2024 vs. FY2025 mix shift comparison, and a margin-vs-mix scatter plot highlighting how product mix drives overall gross margin[cite: 4].
- **Page 3 — Sales Reps (`Who is selling it?`):** Built for National Sales Manager Bongani Twala's Monday review—tracking rep revenue against properly aggregated monthly targets (`Rep Attainment`), territory performance, top accounts, and rep visit activity vs. revenue generated[cite: 2, 4].
- **Page 4 — Receivables (`Where is our cash?`):** Dynamic as-at debtors aging matrix (`Current`, `1-30`, `31-60`, `61-90`, `90+`) split by channel, monthly settlement duration trend, and an actionable collections call list ranked by open balance[cite: 4, 22].

*(Tip: Take screenshots of your 4 Power BI report pages and paste them right here on GitHub just like Amanda did!)*

📁 **Deliverables:**
- [`Sizwe_Commercial_Thandolwethu.pbix`](./Sizwe_Commercial_Thandolwethu.pbix) — Complete Power BI Desktop model & 4-page dashboard[cite: 4, 5]
- [`Sizwe_Executive_Memo_Thandolwethu.pdf`](./Sizwe_Executive_Memo_Thandolwethu.pdf) — One-page executive memo to Financial Director Nomsa Dlamini[cite: 2, 5]
- [`Sizwe_Model_Note_Thandolwethu.pdf`](./Sizwe_Model_Note_Thandolwethu.pdf) — One-page data model architecture note[cite: 5]

---

## 🔍 Key Findings

1. **Material Slowdown in Customer Payment Turnaround Starting January 2024:** Between July and December 2023, `Weighted Days to Settle` remained stable between **36.1 and 37.4 days** (averaging **36.9 days**)[cite: 23]. Starting in **January 2024 (`38.5` days)**, settlement duration climbed steadily for twelve consecutive months to peak at **63.1 days in December 2024**—an increase of **26.2 days** that left **R88.6M** in open receivables as of 30 June 2025[cite: 2, 22, 24, 25].
2. **Weighted Days to Settle (`47.3` Days) vs. Simple Average (`40.9` Days):** Across the full two-year period, `Weighted Days to Settle` (**47.3 days**) sits **6.4 days higher** than unweighted `Avg Days to Settle` (**40.9 days**), with the gap widening to **8.4 days** at the December 2024 peak (`63.1` vs. `54.7` days)[cite: 22, 25]. Because high-volume Subscription accounts settle quickly on small invoice values, the unweighted average masks true working capital lock-up in large Wholesale accounts[cite: 2, 5].
3. **Right-Censored Distortion in Late Q4 FY2025 (April – June 2025):** Settlement duration drops sharply to **29.1 days in April 2025**, **5.0 days in May 2025**, and **3.7 days in June 2025**[cite: 26]. This is a statistical artifact rather than a collections recovery: because `Days to Settle` only evaluates paid invoices, recent slow-paying invoices still open at period-end (`settlement_date` is blank) are excluded from the average[cite: 4, 5]. Pairing the trend line with **Open Balance (`R88.6M`)** and **Open Invoices** prevents misreading period-end cash collection[cite: 4, 5, 22].

---

## 💡 Recommendations & Conclusion

- **Standardize Executive Reporting on Weighted Days to Settle:** Adopt `Weighted Days to Settle` (`47.3` days) as the primary board KPI so working capital reporting reflects actual Rand exposure rather than invoice count[cite: 4, 5, 22].
- **Target High-Exposure Wholesale Accounts in Collections:** Focus immediate credit-control outreach on the top overdue Wholesale accounts identified in the Page 4 Collections Call List to release the **R88.6M** locked in open balances[cite: 2, 4, 22].
- **Monitor Open Balances Alongside Settlement Velocity at Month-End:** Always review recent monthly settlement averages alongside open invoice counts so right-censored period-end cohorts do not create false confidence in cash collection speed[cite: 4, 5].

---

## 📬 References

🔗 [Microsoft DAX Function Reference](https://learn.microsoft.com/dax/)  
📘 [Power BI Star Schema Guidance](https://learn.microsoft.com/power-bi/guidance/star-schema)

---

## ⭐ Let's Connect

- 📧 **Email:** [H_thandolwethu@icloud.com]

I’m always open to feedback, analytics discussions, or collaboration opportunities.
