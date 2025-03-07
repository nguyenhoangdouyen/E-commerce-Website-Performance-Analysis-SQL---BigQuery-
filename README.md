# E-commerce Website Performance Analysis (SQL - BigQuery)  #
## **🎯 I. Introduction** ##
This project analyzes an E-commerce dataset using SQL in Google BigQuery, based on Google Analytics data. It focuses on extracting key business insights, including user behavior, revenue trends, bounce rates, cohort analysis, and purchase patterns. 
## **🎯 II. Requirements** ##
Based on stakeholder requirements, develop 08 BigQuery queries using the Google Analytics dataset to fulfill key responsibilities of a data analyst, ensuring the expected output aligns with business needs.
## **🎯 III. Dataset** ##
The eCommerce dataset is publicly available in Google BigQuery. Follow these steps to access it:

1. Log in to your **Google Cloud Platform** account and create a new project.
2. Open the **BigQuery Console** and select your project.
3. Click **"Add Data"** in the navigation panel, then choose **"Search a project"**.
4. In the search bar, enter the project ID: `bigquery-public-data.google_analytics_sample.ga_sessions` and press **Enter**.
5. Click on the `ga_sessions_` table to explore its structure and data.
## **IV. Exploring the Dataset** ##
***🔍 Query 01: Calculate total visit, pageview, transaction and revenue for January, February and March 2017 (order by month).***
#### 🚀 Queries ####
```sql
SELECT 
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
  SUM(totals.visits) AS total_visit,
  SUM(totals.pageviews) AS total_pageview,
  SUM(totals.transactions) AS total_transaction
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/42eea66b-3f63-46af-9a9f-87d4033262a0)

***🔍 Query 02: Calculate bounce rate traffic source in July 2017).***
#### 🚀 Queries ####
```sql
SELECT 
  trafficSource.source AS source,
  SUM(totals.visits) AS total_visit,
  SUM(totals.bounces) AS total_no_of_bounce,
  ROUND((SUM(totals.bounces) / SUM(totals.visits)) * 100, 3) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visit DESC;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/ca3271d7-aeb2-4857-afcf-df9ce4879727)

***🔍 Query 03: Calculate revenue by traffic source by week, by month in June 2017***
#### 🚀 Queries ####
```sql
WITH month_type AS (
  SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS time,
    trafficSource.source AS source,
    ROUND(SUM(products.productRevenue) / 1000000, 4) AS revenue,
    'Month' AS time_type
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`, 
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS products
  WHERE products.productRevenue IS NOT NULL
  GROUP BY source, time
),
week_type AS (
  SELECT 
    FORMAT_DATE('%Y%W', PARSE_DATE('%Y%m%d', date)) AS time,
    trafficSource.source AS source,
    ROUND(SUM(products.productRevenue) / 1000000, 4) AS revenue,
    'Week' AS time_type
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`, 
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS products
  WHERE products.productRevenue IS NOT NULL
  GROUP BY source, time
)
SELECT time_type, time, source, revenue
FROM month_type
UNION ALL
SELECT time_type, time, source, revenue
FROM week_type
ORDER BY revenue DESC;
```
#### 💡 Queries result ####

![Image](https://github.com/user-attachments/assets/3ae1e18e-72f3-4b45-91e0-cee2ee42d762)

***🔍 Query 04: Calculate average number of pageviews by purchaser type***
#### 🚀 Queries ####
```sql
WITH 
purchaser_data AS (
  SELECT
      FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
      SUM(totals.pageviews) / COUNT(DISTINCT fullvisitorid) AS avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND totals.transactions >= 1
    AND product.productRevenue IS NOT NULL
  GROUP BY month
),
non_purchaser_data AS (
  SELECT
      FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
      SUM(totals.pageviews) / COUNT(DISTINCT fullvisitorid) AS avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND totals.transactions IS NULL
    AND product.productRevenue IS NULL
  GROUP BY month
)
SELECT
    pd.*,
    avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data USING(month)
ORDER BY pd.month;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/fd384e5f-6e4d-4807-bf66-a35ba2b19a72)

***🔍 Query 05: Calculate average number of transactions per user that made a purchase in July 2017***
#### 🚀 Queries ####

```sql
SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    SUM(totals.transactions) / COUNT(DISTINCT fullvisitorid) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) hits,
    UNNEST(product) product
WHERE totals.transactions >= 1
AND product.productRevenue IS NOT NULL
GROUP BY month;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/6c85fb89-5f02-472d-b652-3400b1447279)

***🔍 Query 06: Calculate average amount of money spent per session. Only include purchaser data in July 2017***
#### 🚀 Queries ####
```sql
SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    ((SUM(product.productRevenue)/SUM(totals.visits))/POWER(10,6)) AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) hits,
    UNNEST(product) product
WHERE product.productRevenue IS NOT NULL
AND totals.transactions >= 1
GROUP BY month;
```
#### 💡 Queries result ####

![Image](https://github.com/user-attachments/assets/4ed51427-3666-4b31-8119-a9c2baa565b2)

***🔍 Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.***
#### 🚀 Queries ####
```sql
WITH Purchasers AS (
  SELECT DISTINCT fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.*`, UNNEST(hits) AS hits, UNNEST(hits.product) AS product
  WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
)
SELECT
  product.v2ProductName AS product_name,
  COUNT(*) AS purchase_count
FROM `bigquery-public-data.google_analytics_sample.*`, UNNEST(hits) AS hits, UNNEST(hits.product) AS product
WHERE fullVisitorId IN (SELECT fullVisitorId FROM Purchasers)
AND product.v2ProductName <> "YouTube Men's Vintage Henley"
GROUP BY product_name
ORDER BY purchase_count DESC;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/4caaaa73-dcc7-4b2f-9882-44562bfa6848)

***🔍 Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.***
#### 🚀 Queries ####
```sql
WITH product_data AS (
    SELECT
        FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
        COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
        COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
        COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue IS NOT NULL THEN product.v2ProductName END) AS num_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
         UNNEST(hits) AS hits,
         UNNEST(hits.product) AS product
    WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
          AND eCommerceAction.action_type IN ('2', '3', '6')
    GROUP BY month
    ORDER BY month
)

SELECT
    *,
    ROUND(num_add_to_cart / num_product_view * 100, 2) AS add_to_cart_rate,
    ROUND(num_purchase / num_product_view * 100, 2) AS purchase_rate
FROM product_data;
```
#### 💡 Queries result ####
![Image](https://github.com/user-attachments/assets/9075c9e7-cd9b-4750-a4e7-5df7ba227bd5)

## **🎯 V. How to Run the Queries** ##
1. Open Google BigQuery.
2. Load the dataset bigquery-public-data.google_analytics_sample.
3. Execute each query individually.

## **🎯 VI. How to Run the Queries** ##
This project highlights the application of SQL to analyze web traffic data in Google BigQuery. By writing queries, I explored key metrics such as visits, pageviews, bounce rates, transactions, and revenue per traffic source. These insights help understand user engagement and conversion trends. While SQL provides valuable data extraction and transformation capabilities, further visualization using tools like Power BI or Tableau would enhance the interpretation of these findings.
