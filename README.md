# Customer Lifetime Value (CLV) Cohort Analysis and Forecasting

## Project Overview & Strategic Business Goal
The primary objective of this project is to develop a refined, non-linear Customer Lifetime Value (CLV) model utilizing weekly cohort analysis. Simple corporate metrics (such as historical averages) often fail because they skew actual consumer trajectories and ignore the traffic overhead of non-purchasing visitors. 

To address these limitations, this framework implements a 12-week lifecycle analysis that groups **all website visitors (not just buyers)** by their initial registration week. By coupling multi-layered data joins in SQL with cumulative growth forecasting algorithms in Google Sheets, this project establishes a resilient pipeline to predict long-term user monetization and uncover seasonal acquisition trends.

## Project Constraints & Modeling Rules
- **Data Source:** Google BigQuery clickstream ecosystem (`turing_data_analytics.raw_events` table).
- **Temporal Anchor Window:** Current validation week set to `2021-01-24` (the terminal data entry point).
- **Visitor Inclusion Layer:** Captures **100% of unique user sessions** based on `user_pseudo_id`, incorporating non-purchasing traffic to present an accurate, full-funnel Average Revenue per User (ARPU) baseline.
- **Lifecycle Boundary:** Capped at a strict **12-week analytical boundary** to mirror practical e-commerce churn curves.

---

## Data Infrastructure & SQL Engineering Architecture

The calculation logic is split across three isolated Common Table Expressions (CTEs) designed to accurately compile metrics while bypassing data amplification errors:

```sql
WITH registration_cohort AS (
    -- Step 1: Isolate the absolute first arrival week for 100% of website visitors
    SELECT
        user_pseudo_id,
        TIMESTAMP_TRUNC(TIMESTAMP_MICROS(event_timestamp), WEEK) AS registration_week
    FROM
        `tc-da-1.turing_data_analytics.raw_events`
    QUALIFY 
        ROW_NUMBER() OVER(PARTITION BY user_pseudo_id ORDER BY event_timestamp ASC) = 1
),

revenue_data AS (
    -- Step 2: Extract weekly transactional values and structure date dimensions
    SELECT
        user_pseudo_id,
        TIMESTAMP_TRUNC(TIMESTAMP_MICROS(event_timestamp), WEEK) AS purchase_week,
        SUM(value_in_usd) AS weekly_revenue,
        COUNT(DISTINCT purchase_id) AS total_orders
    FROM
        `tc-da-1.turing_data_analytics.raw_events`
    WHERE
        event_name = 'purchase'
    GROUP BY
        user_pseudo_id, 
        purchase_week
)

-- Step 3: Outer join traffic streams with commerce records to evaluate performance
SELECT
    c.registration_week,
    COUNT(DISTINCT c.user_pseudo_id) AS cohort_user_count,
    FLOOR(TIMESTAMP_DIFF(r.purchase_week, c.registration_week, DAY) / 7) AS relative_week_index,
    SUM(r.weekly_revenue) AS total_segment_revenue
FROM
    registration_cohort c
LEFT JOIN
    revenue_data r ON c.user_pseudo_id = r.user_pseudo_id
GROUP BY
    1, 3
ORDER BY
    c.registration_week ASC, 
    relative_week_index ASC;
```

---

## Analytics Dashboard & Predictive Modeling (Google Sheets)

The downstream architecture utilizes a **3-Table Visual Ledger** with automated conditional heatmaps to trace value maturation over time:

### Table 1: Discrete Average Revenue Per User (ARPU) Matrix
- **Calculation Formula:** $\text{ARPU} = \frac{\text{Total Segment Revenue in Week } n}{\text{Total Users Appended to Registration Cohort}}$
- **Strategic Utility:** Isolates real-time, non-cumulative monetization spikes across user groups.

### Table 2: Cumulative User Value Tracking & Baseline Growth %
- **Calculation Formula:** Maps a continuous running total based on Table 1 ($\text{Cumulative ARPU}_n = \text{ARPU}_n + \text{Cumulative ARPU}_{n-1}$).
- **Core Row Metrics:** 
  - *Cumulative Column Average:* Establishes the centralized benchmark valuation curve for historical cohorts.
  - *Cumulative Growth % (Bottom Row):* Quantifies week-over-week growth progression ($Growth\% = \frac{\mu_{n} - \mu_{n-1}}{\mu_{n-1}}$), mapping the natural deceleration of older user cohorts.

### Table 3: Full 12-Week Extrapolated Matrix & Predictive Forecasting
- **Strategic Challenge:** Recent cohorts lack mature data (e.g., the `2021-01-24` entry only possesses Week 0 metrics at `$0.19`).
- **Forecasting Engine:** Applies historical *Cumulative Growth %* values to project missing weeks, filling empty data arrays across the matrix:
  $$\text{Predicted Value}_{n} = \text{Current Value}_{n-1} \times (1 + \text{Average Cumulative Growth}\%_n)$$
- **Terminal Optimization Insight:** Compiles a unified average across all 12th-week rows to establish an objective, highly reliable full-funnel **Customer Lifetime Value (CLV)** metric that includes non-buyers.

---

## Core Strategic Insights & Executive Trends

1. **Monetization Deceleration:** Cumulative revenue velocity peaks between Week 0 and Week 3, followed by a gradual decay curve. This proves that retention initiatives must be heavily front-loaded within the first 21 days of user acquisition.
2. **Siloed Seasonality:** Comparing the horizontal row gradients reveals distinct acquisition shifts, pointing directly to high-performing marketing channels or successful seasonal promotions.
3. **True North CLV Validation:** The 12th-week cumulative average represents the true dollar boundaries a marketer can spend to acquire a raw user (Max Allowable CAC), preventing overspending on poor-quality traffic.

##  Repository Contents
- `cohort_clv_extraction.sql`: Production-ready SQL query script optimized for Google BigQuery execution.
- `clv_forecasting_dashboard.xlsx`: Source documentation detailing the 3-Table Google Sheets visual workflow, including conditional formatting scripts and predictive trend matrices.

