# Explore Ecommerce Dataset - SQL #
## **I. Introduction** ##
This project analyzes an E-commerce dataset using SQL in Google BigQuery, based on Google Analytics data. It focuses on extracting key business insights, including user behavior, revenue trends, bounce rates, cohort analysis, and purchase patterns. 
## **II. Requirements** ##
Based on stakeholder requirements, develop 08 BigQuery queries using the Google Analytics dataset to fulfill key responsibilities of a data analyst, ensuring the expected output aligns with business needs.
## **III. Dataset** ##
The eCommerce dataset is publicly available in Google BigQuery. Follow these steps to access it:

1. Log in to your **Google Cloud Platform** account and create a new project.
2. Open the **BigQuery Console** and select your project.
3. Click **"Add Data"** in the navigation panel, then choose **"Search a project"**.
4. In the search bar, enter the project ID: `bigquery-public-data.google_analytics_sample.ga_sessions` and press **Enter**.
5. Click on the `ga_sessions_` table to explore its structure and data.
## **IV. Exploring the Dataset** ##
Query 01: Calculate total visit, pageview, transaction and revenue for January, February and March 2017 (order by month).
## Answer ##
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
