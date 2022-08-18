# CLV analysis


✨ ✨ Solution for CLV analysis task for Turing College✨ ✨ 

<a href = 'https://docs.google.com/spreadsheets/d/1lfII6i4Dyt2LiCcsiN8BczGz_HTDvNpZvaFHHD_RAJc/edit?usp=sharing
'> The dashboard with visualisations.</a> It explains the main insights of the analysis and provides a solution for creating CLV analysis.

:rocket: SQL (BigQuery), Excel, Google Sheets, CLV analysis

### The task:
As the first step you should write 1 or 2 queries to pull data of weekly revenue divided by registrations. Since in this particular site there is no concept of registration, we will simply use the first visit to our website as registration date (registration cohort). Do not forget to use user_pseudo_id to distinguish between users. Then divide revenue in consequent weeks by the number of weekly registration numbers.

Next you will produce the same chart, but the revenue / registrations for a particular week cohort will be expressed as a cumulative sum. For this you simply need to add previous week revenue to current week’s revenue. Down below you will calculate averages for all week numbers (weeks since registration).

Basically, the chart above gives you growth of revenue by registered users in cohort for n weeks after registration. While numbers below summarize those values in monetary terms (red marked numbers) and percentage terms (green marked numbers). This provides you with a coherent view of how much revenue you can expect to grow based on your historical data.

Next, we will focus on the future and try to predict the missing data. In this case missing data is the revenue we should expect from later acquired user cohorts. For example, for users whose first session happened on 2021-01-24 week we have only their first week revenue which is 0.19USD per user who started their first session in this week. Though we are not sure what will happen in the next 12 weeks.

For this we will simply use previously calculated Cumulative growth % (red marked area in chart aboove) and predict all 12 future weeks values (ex. for this cohort we can calculate expected revenue for week 1 as 0.19USD x (1 + 23.29%) = 0.24USD, for week 2 as 0.24USD x (1 + 12.26%) = 0.27USD). Using Avg. cumulative growth for each week we can calculate that based on 0.19USD initial value we can expect 0.35USD as revenue on week 12. Provide a chart which calculates these numbers for all future weeks (up till week 12).

ou should calculate the average of cumulative revenue for 12th week for all users who have been on your website. This not only provide better estimate of CLV for all your users who have been on your website (including the ones who did not purchase anything) but also allows you to see trends for weekly cohorts. Have a look at the conditional formatting you added to all 3 charts and be prepared to answer follow up questions.

### Main SQL Query:

``` SQL 
WITH t1 AS 
(
  SELECT
  DISTINCT user_pseudo_id,
  MIN(DATE_TRUNC(PARSE_DATE("%Y%m%d", event_date), WEEK)) as start_week
  FROM `tc-da-1.turing_data_analytics.raw_events`
  GROUP BY 1
),


t2 AS 
(
  SELECT re.user_pseudo_id, 
    DATE_TRUNC(PARSE_DATE("%Y%m%d", event_date), WEEK) AS purchase_date, 
    purchase_revenue_in_usd AS revenue
  FROM `tc-da-1.turing_data_analytics.raw_events` AS re
  LEFT JOIN t1 AS c
  ON c.user_pseudo_id = re.user_pseudo_id
)


SELECT t1.start_week,
  SUM(CASE WHEN t2.purchase_date = start_week THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w0,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 1 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w1,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 2 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w2,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 3 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w3,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 4 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w4,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 5 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w5,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 6 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w6,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 7 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w7,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 8 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w8,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 9 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w9,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 10 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w10,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 11 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w11,
  SUM(CASE WHEN t2.purchase_date = DATE_ADD(start_week, INTERVAL 12 WEEK) THEN revenue END)/COUNT(DISTINCT t1.user_pseudo_id) AS w12
FROM t1
JOIN t2
ON t1.user_pseudo_id = t2.user_pseudo_id
GROUP BY 1
ORDER BY 1
```
