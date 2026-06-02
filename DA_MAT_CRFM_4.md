# CLV Task

Your manager has requested a more refined Customer Lifetime Value (CLV) analysis after reading that Shopify's formula is too simplistic. He believes cohort analysis provides more reliable and actionable insights. Here is key context information before starting:

- Use the [turing_data_analytics.raw_events](https://console.cloud.google.com/bigquery?authuser=3&project=tc-da-1&ws=!1m14!1m3!8m2!1s756497109418!2s1b8fa7cb3d73401f81fea1591fa615a7!1m4!4m3!1stc-da-1!2sturing_data_analytics!3sstackoverflow_posts!1m4!4m3!1stc-da-1!2sturing_data_analytics!3sraw_events&d=turing_data_analytics&p=tc-da-1&t=raw_events&page=table) table for analysis.
- Assume the current week is 2021-01-24 (the last weekly cohort in the dataset).

The main manager's identified issues with previous analysis are:

- Previously, only purchasing customers were included. But marketing considers all users who visit the website, not just buyers. Adjust the analysis to include all visitors, not just purchasers.
- Customers do not tend to stay engaged for long. The manager wants a **weekly cohort analysis** for a 12-week period. Customers should not be expected to stay beyond 12 weeks.

To address these issues, we will track users by their **first event week** (let's call it registration cohort from now on) and analyze their spending over 12 weeks. The following steps might help you to achieve this goal:

1. Registration Cohort Query/CTE:

    - Extract each user's **first event week** (registration week).
    - Use `user_pseudo_id` to distinguish between users.

2. Revenue Data Query/CTE: Extract **weekly user purchases** and set the date format to be the same as the registration cohort.

3. Join Registration Cohort with Revenue Data

4. Calculate Cohort-Based CLV: Divide revenue from subsequent weeks by the number of users in the registration cohort, to get Average Revenue per User (ARPU) for each cohort.

Once you apply conditional formatting to your data, the end result should look something like this:

<div><img src="https://i.imgur.com/Io8Lc2p.jpg" /></div>

Now, you will generate the same chart, but with weekly average revenue for each cohort expressed as a cumulative sum, following these steps:

- Add each week's revenue to the previous week's revenue to create a **running total**.
- Ensure the cumulative calculations are based on the **first table's weekly averages** so the new cumulative cohort table accurately reflects these values.

The initial chart tracks **how revenue grows over time** for users in a given cohort, monitoring spending over n weeks after registration. The **cumulative version** summarizes revenue growth over time, allowing for better forecasting and trend analysis. To summarise:

- Each row should represent a cohort based on registration week, tracking their cumulative Average Revenue per User over time.
- The **Cumulative Column Average** row shows the overall average revenue per cohort each week.
- **Cumulative Growth %** (bottom row) indicates week-over-week revenue growth, which gradually slows down over time.

<div><img src="https://i.imgur.com/fX37psr.jpg" /></div>

Next, we will focus on the future and try to predict the missing data. In this case, missing data is the revenue we should expect from later acquired user cohorts. For example, for users who registered on 2021-01-24 week, we have only their first-week revenue, which is 0.19 USD per user. Hence, we are not sure what will happen in the next 12 weeks.

For this, we will simply use previously calculated Cumulative growth % and predict all 12 future weeks' values (ex., for this cohort, we can calculate expected revenue for week 1 as 0.19 USD x (1 + 23.29 %) = 0.24 USD, for week 2 as 0.24 USD x (1 + 12.26%) = 0.27 USD). Using average cumulative growth for each week, we can calculate that based on the 0.19 USD initial value, we can expect 0.35 USD as revenue on week 12. Provide a chart that calculates these numbers for all 12 future weeks. See the example chart below, which should show the following numbers for all weekly user cohorts:

<div><img src="https://i.imgur.com/oPgc6kx.jpg" /></div>

The chart above is our final result. You should calculate the average of cumulative revenue for the 12th week for all users who have been on your website. This not only provides a better estimate of CLV for all your users who have been on your website (including the ones who did not purchase anything), but it also allows you to see trends for weekly cohorts.

When writing insights from these 3 cohort analysis tables, focus on identifying key trends, such as revenue growth patterns, seasonal effects, and long-term customer value. Have a look at the conditional formatting you added to all three charts, and be prepared to answer follow-up questions.

<br>

## Evaluation Criteria for Graded Project Submission

- SQL: Correct columns identified to make analysis
- SQL: Correctly calculated main metrics: revenue, orders, customers
- SQL: Correctly calculated user counts by weekly cohorts
- Google Sheets: Visualization follows the examples above; at least three different tables with conditional formatting applied are provided
- Analysis: Findings and calculations are correct and the main trends identified
- Analytical approach to the problem

<br>

During a task review, you may get asked questions that test your understanding of covered topics.

### Sample questions:
- Does including all customers (not only the ones who purchased something) influence the calculation of CLV?
- Is your previously calculated CLV (using Shopify formula) still accurate?
- How does cohort analysis help you understand what revenue you can expect in the future?
- If we knew that our business Take Rate (% of the revenue we can actually treat as profit) is only 10%, would you use this percentage to estimate the Lifetime value of the customer? Could you compare it against customer acquisition costs provided by marketing?
- What caveats can you expect from such an approach?

<br>

## Submission

If you used Power BI, please upload your Dashboard example and retrieve it as a PDF file to your project repository.

> Upload Spreadsheets or Excel solutions and **.sql** files to the GitHub repository for both this task and the RFM graded task (from part 1). Prepare to present both projects during your project review.

