# 💻 E-commerce Website Performance Analysis (SQL - BigQuery) 

![image](https://github.com/user-attachments/assets/4a8401e3-ac9f-44db-aa80-60f83015406a)

**Author:** Nguyen Hoang Do Uyen

**Date:** 07/10/2001 

**Tools Used:** SQL

## 📑 Table of Contents

[📌 Background & Overview](#-background--overview)  
[📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
[🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)  

## Background & Overview

### 📖 What is this project about? What Business Question will it solve?  

This project uses **SQL** and **BigQuery** to analyze the performance of an e-commerce website, focusing on traffic, engagement, and revenue over time.  
- ✔️ **Track changes** in traffic, engagement, and revenue.  
- ✔️ **Breakdown revenue** by traffic source (weekly, monthly).  
- ✔️ **Compare pageviews** and transactions between purchasers and non-purchasers.  
- ✔️ **Map the customer journey** from product view to purchase.

## 👤 Who is this project for?  
- ✔️ **Data Analysts & Business Analysts**  
- ✔️ **Digital Marketing Teams**  
- ✔️ **E-commerce Managers & Stakeholders**  
- ✔️ **Business Intelligence Teams**

## 📂 Dataset Description & Data Structure

** 📌 Data Source**: The sample data is from **Google Analytics 4 (GA4)**, exported to **BigQuery**, including user activity data from the **Google Merchandise Store** e-commerce website.

**📌 Data Size**:

- **Dataset**: `ga4_obfuscated_sample_ecommerce`

**📌 How to Access the Data:**
1. Log in to your **Google Cloud Platform** account and create a new project.
2. Open the **BigQuery Console** and select your project.
3. Click on **"Add Data"** in the navigation panel, then choose **"Search a project"**.
4. In the search bar, enter the project ID: `bigquery-public-data.google_analytics_sample.ga_sessions` and press **Enter**.
5. Click on the `ga_sessions_` table to explore its structure and data.

## ⚒️ Main Process

### 🔍 Calculate total visit, pageview, transaction and revenue for January, February and March 2017 (order by month).

The goal of this analysis is to calculate the **total number of visits, pageviews, transactions, and revenue** for each month (January, February, and March) in 2017, based on the data provided. The results will be ordered by month to observe the trends and variations in website performance over these three months.

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

### 🔍 Calculate bounce rate traffic source in July 2017).

The goal of this analysis is to calculate the **bounce rate** for different **traffic sources in July 2017**. By analyzing bounce rates across various traffic sources, we aim to identify which sources lead to higher user engagement and which may need optimization to improve user retention and website performance.

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

### 🔍 Calculate revenue by traffic source by week, by month in June 2017

This analysis is to calculate the **revenue** generated by different **traffic sources** for **June 2017**, broken down by **week** and **month**. This analysis will help identify the most profitable traffic sources over time, allowing for targeted marketing and optimization strategies to maximize revenue growth.

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

### 🔍 Calculate average number of pageviews by purchaser type

Calculate the **average number of pageviews** by **purchaser type** to understand how user behavior differs between purchasers and non-purchasers, providing insights into user engagement and purchase behavior.

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

### 🔍 Calculate average number of transactions per user that made a purchase in July 2017

Calculate the **average number of transactions per user** who made a purchase in **July 2017** to explore the purchasing patterns and frequency of repeat buyers. This will provide a clearer understanding of customer loyalty and help identify opportunities to enhance retention strategies.

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

### 🔍 Calculate average amount of money spent per session. Only include purchaser data in July 2017

Determine the **average amount of money spent per session** by **purchasers in July 2017**, focusing on those who made a purchase. This analysis will offer insights into customer spending patterns, helping to evaluate marketing strategy effectiveness and identify high-value customer segments.

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

### 🔍 Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.

Identify the **other products purchased** by customers who bought the **"YouTube Men's Vintage Henley"** in **July 2017**. The output should list the **product name** and the **quantity ordered**, providing insights into cross-selling opportunities and customer preferences.

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

### 🔍 Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.

Generate a **cohort map** to track the customer journey from **product view** to **add-to-cart** and finally to **purchase** in **January, February, and March 2017**. This will help analyze conversion rates at each stage of the purchase funnel, providing insights into where customers drop off and identifying opportunities to optimize the sales process.

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

## 🔎 Final Conclusion & Recommendations

### 📌 Insights

- **Traffic Trends**: The **total visits** in January, February, and March 2017 showed consistent traffic, with March having the highest number of visits (69,931).
- **Revenue by Source**: **Direct** traffic contributed significantly to revenue in June 2017, with notable contributions from **Google** and **Mail** sources, indicating strong performance from these channels.
- **Bounce Rate Insights**: Traffic from sources like **phandroid.com** has a high bounce rate of **77.78%**, which suggests that users might not be engaging well with the content on landing pages.
- **Purchaser Behavior**: Customers who bought the **"YouTube Men's Vintage Henley"** showed interest in related products like **Google Men's Vintage Badge Tee Black**, which presents an opportunity for cross-selling.

### 📌 Recommendations

1. **Optimize Landing Pages for High Bounce Traffic**: Improve pages for sources with high bounce rates (e.g., **phandroid.com**) by enhancing content relevance and reducing page load times.
2. **Focus on Direct & Paid Traffic**: Increase investment in **Google Ads** and improve brand awareness to leverage high-performing **direct traffic** for better conversions.
3. **Boost Conversion for Non-Purchasers**: Simplify the checkout process and offer targeted promotions to convert high-pageview, non-purchasing users.
4. **Maximize Cross-Selling**: Use product recommendations for items like the **"YouTube Men's Vintage Henley"** to drive cross-sales and increase average order value.

