# Healthcare Denial Management Dashboard

## 1. Project Overview

### Description

Claim denials are one of the biggest sources of revenue leakage in healthcare revenue cycle management. Every denied claim represents delayed or lost cash, extra rework for billing staff, and a ticking clock against payer appeal deadlines. Left untracked, denials pile up across payers, providers, and denial reasons with no clear view of where the money is actually stuck or which claims are still recoverable.

This project builds an end-to-end **Denial Management Dashboard** in Power BI that turns raw claims and payer-rule data into an actionable revenue cycle tool. It answers the core business questions revenue cycle teams face every month:

- How much are we billing, and how much of that is being denied, paid, or still pending?
- Which payers, denial reasons, and provider specialties are driving the most denials?
- How much money is currently "at risk," and how much of it is still actionable before an appeal deadline passes?
- Are our appeals working — what's our appeal rate and overturn rate?
- Which specific claims need a biller's attention right now?

The goal is to move the team from reactive, spreadsheet-based denial tracking to a proactive, drillable dashboard that surfaces where to focus recovery efforts first.

### Target Audience

- **Revenue Cycle Managers / Directors** — monitor denial rate, money at risk, and recovery trends at a portfolio level.
- **Denial & Appeals Analysts** — drill into specific denial categories, denial codes, and payers to prioritize working the queue.
- **Medical Billers / Claims Specialists** — use the Claims drill-through page to find and work individual unworked claims before their appeal deadline expires.
- **Payer Relations / Contracting teams** — review payer-level denial and recovery performance to support payer negotiations.
- **Finance / Executive stakeholders** — track high-level KPIs (denial rate, net collection rate, money at risk) against prior-year and target benchmarks.

## 2. Key Features & KPIs

### Metrics

The model includes a dedicated set of DAX measures covering volume, financial exposure, timeliness, and appeal performance:

**Volume & Claims**
- Denied Claims Count
- Total Claims Unworked
- Overturned Claims Count

**Financial**
- Billed Amount (current period, with prior-year comparison — *Billed Amount CF / Billed Amount vs PY*)
- Denied Amount (current period, with prior-year comparison — *Denied Amount CF / Denied Amount vs PY*)
- Paid Amount (current period, with prior-year comparison — *Paid Amount CF / Paid Amount vs PY*)
- Pending Amount (current period, with prior-year comparison — *Pending Amount CF / Pending Amount vs PY*)
- Net Recovery (current period, with prior-year comparison — *Net Recovery CF / Net Recovery vs PY*)
- Money At Risk / Money At Risk CF
- Actionable Unworked Revenue
- Total Pending Revenue CF

**Rates & Ratios**
- Denial Rate % (current period, prior-year, and vs. Target — *Denial Rate CF, Denial Rate Target, Denied Rate PY CF, Denied Rate vs PY*)
- Avoidable Denial %
- Net Collection Rate %
- Appeal Rate %
- Appeal Overturn Rate %

**Timeliness**
- Avg Days to Pay
- Avg Days Since Denial
- Days Until Appeal Deadline
- Deadline Urgency Bucket (categorizes claims by how close they are to losing their appeal window)

### Interactivity

The report is built as a 5-page interactive experience:

| Page | Purpose |
|---|---|
| **Overview** | Executive summary of denial rate, financial KPIs, and trends vs. prior year/target |
| **Denial** | Breaks down denials by category, denial code/reason, and provider specialty |
| **Payer** | Compares denial and recovery performance across payers |
| **Appeals** | Tracks appeal rate, overturn rate, and appeal deadline urgency |
| **Claims** | A drill-through detail page listing individual claims for triage/action |

**Filtering & navigation:**
- **Slicers** for Year, Quarter, and Month Name (Date table) are available across multiple pages for consistent time-period filtering.
- A **Payer Type** slicer on the Payer page filters the report by payer segment.
- A **Deadline Urgency Bucket** slicer on the Claims page lets billers filter directly to claims that are closest to their appeal deadline.
- A **dynamic sort-metric slicer** (`p_Reason_Sort_Metric`) lets users re-rank denial reason visuals by different measures (e.g., by count vs. by dollar amount).
- **Drill-through** is configured from the Denial and Payer pages into the **Claims** detail page, passing `payer_type`, `denial_category`, `denial_code_description`, and `provider_specialty` as context — so a user can right-click a denial reason or payer and jump straight to the underlying claim-level records.
- **Page navigation buttons** and a **bookmark navigator** are used for guided navigation between pages/views.
- Visuals include cards (KPI tiles), clustered bar/column charts, a funnel, a combo (line + column) chart, a 100% stacked area chart, a matrix/pivot table, and a detailed table for claim-level review.

## 3. Data Source & Architecture

### Sources

- **Kaggle dataset:** [DenialIQ: 120K Medical Claims | X12 Denial Codes](https://www.kaggle.com/) — a synthetic/anonymized dataset of ~120,000 medical claims with associated X12 denial codes, payer information, and claim adjudication details.

### Data Model

The model is built as a **star schema**, with a central claims fact table connected to supporting dimension tables:

- **Claims (fact table)** — one row per claim; holds claim IDs, billed/paid/denied amounts, dates, provider specialty, recovery action, and appeal deadline fields. This is the grain the core KPI measures are calculated against.
- **Denial (dimension)** — denial category and denial code description/reason lookup, related to Claims to enable "denial reason" breakdowns and drill-through filtering.
- **Payers (dimension)** — payer type/segment lookup, related to Claims to enable payer-level comparisons.
- **Date (dimension)** — a standard date table (Year, Quarter, Month Name, Year-Month) used for all time-intelligence measures and slicers, related to the relevant date column(s) on Claims.
- **p_Reason Sort Metric (disconnected/parameter table)** — a "field parameter" style table used purely to let users dynamically switch which measure ranks the denial-reason visuals, without being part of the physical relationships.

Relationships flow from the dimension tables (Denial, Payers, Date) into the Claims fact table (one-to-many), which is the standard star-schema pattern that keeps filters propagating correctly and DAX measures simple and performant.

### ETL Process (Power Query)

The following cleaning and shaping steps were applied in Power Query before loading the model:

- **Null handling:** replaced nulls in `modifier` with `"No Modifier"`, and replaced nulls in `prior_auth_number` with `"N/A"`, so downstream visuals and filters don't drop or misrepresent blank values.
- **Relationship key engineering:** in both `claims_main` and `payer_rules`, created a merged custom column `payer_cpt_key` (built as `payer_type & "-" & cpt_code`) so that a direct one-to-many relationship could be established between claims and payer-specific CPT rules without a composite/multi-column join.
- **Data type optimization:** reviewed and set correct data types (dates as Date, amounts as Decimal/Currency, IDs and codes as Text, categorical fields as Text) to reduce model size and ensure measures/time intelligence behave correctly.
- **Data validation:** checked the cleaned tables for consistency (e.g., row counts, key uniqueness/referential integrity between claims and payer rules, and expected value ranges) before finalizing the load into the model.

## 4. How to Use and Share

### Installation Guide

1. **Prerequisites:** Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free) — required to open and edit the `.pbix` file.
2. **Open the file:** Double-click `Healthcare_Denial_Management_Dashboard.pbix`, or open Power BI Desktop and use **File → Open** to browse to it.
3. **Refresh data (if needed):** If the underlying source data has changed or moved, use **Home → Refresh** to reload the latest data through the existing Power Query steps.


### User Guide

- **Navigate pages:** Use the page tabs at the bottom (or the in-report navigation buttons/bookmark navigator on the Overview page) to move between Overview, Denial, Payer, Appeals, and Claims.
- **Filter by time period:** Use the Year, Quarter, or Month slicers found on each page to narrow every visual on that page down to a specific period.
- **Filter by payer or urgency:** On the Payer page, use the Payer Type slicer; on the Claims page, use the Deadline Urgency Bucket slicer to focus on claims closest to losing their appeal window.
- **Re-rank denial reasons:** Use the dynamic sort-metric slicer on the Denial page to switch the denial-reason charts between ranking by count vs. by dollar value.
- **Drill through to claim detail:** Right-click any denial category, denial reason, or payer value on the Denial or Payer pages and select **Drill through → Claims** to jump to the filtered, claim-level detail table for that selection (useful for billers who need to work specific claims).
- **Cross-filter with visuals:** Click any bar, segment, or category in a chart to cross-filter the rest of the visuals on that page; click again (or use the eraser/reset icon) to clear the filter.
- **Export or share:** Use **File → Export** in Power BI Desktop, or the **Share**/**Export** options in the Power BI Service, to distribute snapshots, PDFs, or live report links to stakeholders.
