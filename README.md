# Ecommerce Project
Use SQL for exploring Ecommerce Dataset
![image](https://github.com/user-attachments/assets/dc267b17-4eb9-4eb4-83bf-b3d329bbe09d)


- Author: Nguyen Thuy Hang
- Date: 2025-06-22
- Tools Used: SQL  

# 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🔎 Exploring dataset & Insights](#-exploring-dataset-&-insights)

---

## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? What Business Question will it solve?

This project includes an eCommerce dataset that I explore by using SQL on Google BigQuery. The data originates from the Google Analytics public dataset and represents information from an eCommerce website.

The main business question this project aims to answer is: How does user behavior and traffic source impact key eCommerce metrics such as visits, pageviews, transactions, bounce rate, revenue, and product purchase patterns over time?



### 👤 Who is this project for?  
This project is for *business analysts, marketing teams, product managers, and data analysts* who need actionable insights into customer behavior, traffic sources, and sales performance to drive smarter business decisions and optimize the eCommerce experience.


---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: the dataset is obtained from Google Analytics public dataset 

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  

There is a table are in the dataset.  

#### 2️⃣ Table Schema

| Field Name | Data Type | Description |
| --- | --- | --- |
| fullVisitorId | STRING | The unique visitor ID. |
| date | STRING | The date of the session in YYYYMMDD format. |
| totals | RECORD | This section contains aggregate values across the session. |
| totals.bounces | INTEGER | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null. |
| totals.hits | INTEGER | Total number of hits within the session. |
| totals.pageviews | INTEGER | Total number of pageviews within the session. |
| totals.visits | INTEGER | The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session. |
| totals.transactions | INTEGER | Total number of ecommerce transactions within the session. |
| trafficSource.source | STRING | The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter. |
| hits | RECORD | This row and nested fields are populated for any and all types of hits. |
| hits.eCommerceAction | RECORD | This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected. |
| hits.eCommerceAction.action_type | STRING | The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0.<br>Usually this action type applies to all the products in a hit, with the following exception: when hits.product.isImpression = TRUE, the corresponding product is a product impression that is seen while the product action is taking place (i.e., a "product in list view").<br>Example query to calculate number of products in list views:<br>SELECT<br>COUNT(hits.product.v2ProductName)<br>FROM [foo-160803:123456789.ga_sessions_20170101]<br>WHERE hits.product.isImpression == TRUE<br>Example query to calculate number of products in detailed view:<br>SELECT<br>COUNT(hits.product.v2ProductName),<br>FROM<br>[foo-160803:123456789.ga_sessions_20170101]<br>WHERE<br>hits.ecommerceaction.action_type = '2'<br>AND ( BOOLEAN(hits.product.isImpression) IS NULL OR BOOLEAN(hits.product.isImpression) == FALSE ) |
| hits.product | RECORD | This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data. |
| hits.product.productQuantity | INTEGER | The quantity of the product purchased. |
| hits.product.productRevenue | INTEGER | The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000). |
| hits.product.productSKU | STRING | Product SKU. |
| hits.product.v2ProductName | STRING | Product Name. |


## ⚒️ Main Process

1️⃣ Understand dataset 

2️⃣ Explore specific dataset through some requirements


---

## 🔎 Exploring dataset & insights  

**Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)**

*Purpose*: To measure user interest and behavior in the first three months of the year, thereby determining traffic trends, engagement levels and shopping needs over time
-> Help businesses grasp the overall picture of performance at the beginning of the year, see when traffic and transactions are highest, as a basis for assessing seasonal fluctuations

```sql
SELECT  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month_trans
        ,SUM(totals.visits) AS total_visits
        ,SUM(totals.pageviews) AS total_pageviews
        ,SUM(totals.transactions) AS total_transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix between '0101' AND '0331'
GROUP BY month_trans
ORDER BY month_trans ASC
```
**Result**
| month_trans | total_visits | total_pageviews | total_transactions |
| --- | --- | --- | --- |
| 201701 | 64694 | 257708 | 713 |
| 201702 | 62192 | 233373 | 733 |
| 201703 | 69931 | 259522 | 993 |



**Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)**

*Purpose*: Identify low-quality traffic sources (high bounce rate), then develop a strategy to optimize ad spend or improve target content.

```sql
SELECT  trafficSource.source
        ,SUM(totals.visits) AS total_visits
        ,SUM(totals.bounces) AS num_bounce
        ,ROUND(SUM(totals.bounces)/SUM(totals.visits),3) AS bounce_rates
FROM bigquery-public-data.google_analytics_sample.ga_sessions_20170701
GROUP BY trafficSource.source
ORDER BY trafficSource.source ASC
```
**Result**
| source | total_visits | num_bounce | bounce_rates |
| --- | --- | --- | --- |
| (direct) | 335 | 181 | 0.54 |
| Partners | 17 | 11 | 647 |
| analytics.google.com | 19 | 13 | 684 |
| ask | 2 | 2 | 1 |
| baidu | 8 | 6 | 0.75 |
| bing | 2 | 1 | 0.5 |
| blog.golang.org | 2 | 1 | 0.5 |
| dfa | 1 |  |  |
| docs.google.com | 1 | 1 | 1 |
| facebook.com | 46 | 22 | 478 |
| google | 1172 | 663 | 566 |
| google.com | 7 | 2 | 286 |
| l.facebook.com | 3 | 3 | 1 |
| m.baidu.com | 1 | 1 | 1 |
| m.facebook.com | 255 | 152 | 596 |
| qiita.com | 2 | 1 | 0.5 |
| quora.com | 5 | 5 | 1 |
| reddit.com | 4 | 1 | 0.25 |
| search.xfinity.com | 1 | 1 | 1 |
| t.co | 1 |  |  |
| web.facebook.com | 1 | 1 | 1 |
| yahoo | 1 | 1 | 1 |
| youtube.com | 162 | 121 | 747 |


**Query 3: Revenue by traffic source by week, by month in June 2017**

*Purpose*: Track revenue generated from each traffic source, broken down by time (week/month), to understand which channels actually deliver financial value
-> allows businesses to evaluate financial performance by marketing channel and track short-term revenue fluctuations, serving campaign analysis or optimizing advertising spending

```sql
SELECT
    'Month' AS time_type
    ,FORMAT_DATE('%Y%m', DATE(PARSE_DATE('%Y%m%d', date))) AS time
    ,trafficSource.source AS source
    ,SUM(product.productRevenue) / 1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
,UNNEST(hits) AS hit
,UNNEST(hit.product) AS product
WHERE product.productRevenue IS NOT NULL
GROUP BY time, source

UNION ALL

SELECT
    'Week' AS time_type
    ,FORMAT_DATE('%Y%W', DATE(PARSE_DATE('%Y%m%d', date))) AS time
    ,trafficSource.source AS source
    ,SUM(product.productRevenue) / 1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
,UNNEST(hits) AS hit
,UNNEST(hit.product) AS product
WHERE product.productRevenue IS NOT NULL
GROUP BY time, source

ORDER BY time, revenue desc;
```
**Result**
| time_type | time | source | revenue |
| --- | --- | --- | --- |
| Month | 201706 | (direct) | 97,333.619695 |
| Month | 201706 | google | 18,757.17992 |
| Month | 201706 | dfa | 8,862.229996 |
| Month | 201706 | mail.google.com | 2,563.13 |
| Month | 201706 | search.myway.com | 105.939.998 |
| Month | 201706 | groups.google.com | 101.96 |
| Month | 201706 | chat.google.com | 74.03 |
| Month | 201706 | dealspotr.com | 72.95 |
| Month | 201706 | mail.aol.com | 64.849.998 |
| Month | 201706 | phandroid.com | 52.95 |
| Month | 201706 | sites.google.com | 39.17 |
| Month | 201706 | google.com | 23.99 |
| Month | 201706 | yahoo | 20.39 |
| Month | 201706 | youtube.com | 16.99 |
| Month | 201706 | bing | 13.98 |
| Month | 201706 | l.facebook.com | 12.48 |
| Week | 201722 | (direct) | 6,888.899975 |
| Week | 201722 | google | 2,119.38999 |
| Week | 201722 | dfa | 1,670.649998 |
| Week | 201722 | sites.google.com | 13.98 |
| Week | 201723 | (direct) | 17,325.679919 |
| Week | 201723 | dfa | 1,145.279998 |
| Week | 201723 | google | 1,083.949999 |
| Week | 201723 | search.myway.com | 105.939.998 |
| Week | 201723 | chat.google.com | 74.03 |
| Week | 201723 | youtube.com | 16.99 |
| Week | 201724 | (direct) | 30,908.909927 |
| Week | 201724 | google | 9,217.169976 |
| Week | 201724 | mail.google.com | 2,486.86 |
| Week | 201724 | dfa | 2,341.56 |
| Week | 201724 | dealspotr.com | 72.95 |
| Week | 201724 | bing | 13.98 |
| Week | 201724 | l.facebook.com | 12.48 |
| Week | 201725 | (direct) | 27,295.319924 |
| Week | 201725 | google | 1,006.099991 |
| Week | 201725 | mail.google.com | 76.27 |
| Week | 201725 | mail.aol.com | 64.849.998 |
| Week | 201725 | phandroid.com | 52.95 |
| Week | 201725 | groups.google.com | 38.59 |
| Week | 201725 | sites.google.com | 25.19 |
| Week | 201725 | google.com | 23.99 |
| Week | 201726 | (direct) | 14,914.80995 |
| Week | 201726 | google | 5,330.569964 |
| Week | 201726 | dfa | 3,704.74 |
| Week | 201726 | groups.google.com | 63.37 |
| Week | 201726 | yahoo | 20.39 |



**Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017**

*Purpose*: Understand the differences in interest levels and product discovery behavior of customer groups, thereby helping to identify interaction characteristics related to purchase likelihood.
```sql
with 
purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      (sum(totals.pageviews)/count(distinct fullvisitorid)) as avg_pageviews_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions>=1
  and product.productRevenue is not null
  group by month
),

non_purchaser_data as(
  select
      format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
      sum(totals.pageviews)/count(distinct fullvisitorid) as avg_pageviews_non_purchase,
  from `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,unnest(hits) hits
    ,unnest(product) product
  where _table_suffix between '0601' and '0731'
  and totals.transactions is null
  and product.productRevenue is null
  group by month
)

select
    pd.*,
    avg_pageviews_non_purchase
from purchaser_data pd
full join non_purchaser_data using(month)
order by pd.month;
```
**Result**
| month | avg_pageviews_purchase | avg_pageviews_non_purchase |
| --- | --- | --- |
| 201706 | 940.205.011.389.522 | 316.865.588.463.417 |
| 201707 | 12.423.755.186.722 | 33.405.655.979.568 |



**Query 05: Average number of transactions per user that made a purchase in July 2017**

*Purpose*: Measure customer loyalty and repeat purchase behavior. It is the basis for designing customer retention, upsell, and loyalty programs

```sql
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    sum(totals.transactions)/count(distinct fullvisitorid) as Avg_total_transactions_per_user
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,unnest (hits) hits,
    unnest(product) product
where  totals.transactions>=1
and product.productRevenue is not null
group by month;
```
**Result**
| month | Avg_total_transactions_per_user |
| --- | --- |
| 201707 | 416.390.041.493.776 |

**Query 06: Average amount of money spent per session. Only include purchaser data in July 2017**

*Purpose*: 
- Analyze the value of each converted visit, support ROI calculation for campaigns
- Help set benchmark AOV per session, thereby setting target price for advertising campaigns

```sql
select
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    ((sum(product.productRevenue)/sum(totals.visits))/power(10,6)) as avg_revenue_by_user_per_visit
from `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,unnest(hits) hits
  ,unnest(product) product
where product.productRevenue is not null
  and totals.transactions>=1
group by month;
```
**Result**
| month | avg_revenue_by_user_per_visit |
| --- | --- |
| 201707 | 438.565.983.480.512 |


**Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered**

*Purpose*: Understand cross-sell behavior, support smart product recommendations (recommendation engine).

```sql
with buyer_list as(
    SELECT
        distinct fullVisitorId  
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```
**Result**
| other_purchased_products | quantity |
| --- | --- |
| Google Sunglasses | 20 |
| Google Women's Vintage Hero Tee Black | 7 |
| SPF-15 Slim & Slender Lip Balm | 6 |
| Google Women's Short Sleeve Hero Tee Red Heather | 4 |
| YouTube Men's Fleece Hoodie Black | 3 |
| Google Men's Short Sleeve Badge Tee Charcoal | 3 |
| 22 oz YouTube Bottle Infuser | 2 |
| Android Men's Vintage Henley | 2 |
| Recycled Mouse Pad | 2 |
| Red Shine 15 oz Mug | 2 |
| Google Doodle Decal | 2 |
| YouTube Twill Cap | 2 |
| Google Men's Short Sleeve Hero Tee Charcoal | 2 |
| Android Women's Fleece Hoodie | 2 |
| Android Wool Heather Cap Heather/Black | 2 |
| Crunch Noise Dog Toy | 2 |
| Google Men's 100% Cotton Short Sleeve Hero Tee Red | 1 |
| Android Men's Vintage Tank | 1 |
| YouTube Women's Short Sleeve Hero Tee Charcoal | 1 |
| Google Men's Performance Full Zip Jacket Black | 1 |
| 26 oz Double Wall Insulated Bottle | 1 |
| Android Men's Short Sleeve Hero Tee White | 1 |
| Android Men's Pep Rally Short Sleeve Tee Navy | 1 |
| YouTube Men's Short Sleeve Hero Tee Black | 1 |
| Google Slim Utility Travel Bag | 1 |
| Google Men's Pullover Hoodie Grey | 1 |
| YouTube Men's Short Sleeve Hero Tee White | 1 |
| Google Men's  Zip Hoodie | 1 |
| Google Men's Vintage Badge Tee White | 1 |
| Google Men's Long & Lean Tee Charcoal | 1 |
| YouTube Custom Decals | 1 |
| Four Color Retractable Pen | 1 |
| Google Laptop and Cell Phone Stickers | 1 |
| Google Men's Vintage Badge Tee Black | 1 |
| Google Men's Long Sleeve Raglan Ocean Blue | 1 |
| Google Men's Bike Short Sleeve Tee Charcoal | 1 |
| Google 5-Panel Cap | 1 |
| Google Toddler Short Sleeve T-shirt Grey | 1 |
| Android Sticker Sheet Ultra Removable | 1 |
| Google Twill Cap | 1 |
| Google Men's Long & Lean Tee Grey | 1 |
| YouTube Men's Long & Lean Tee Charcoal | 1 |
| Google Women's Long Sleeve Tee Lavender | 1 |
| YouTube Hard Cover Journal | 1 |
| Android BTTF Moonshot Graphic Tee | 1 |
| Google Men's Airflow 1/4 Zip Pullover Black | 1 |
| Android Men's Short Sleeve Hero Tee Heather | 1 |
| YouTube Women's Short Sleeve Tri-blend Badge Tee Charcoal | 1 |
| Google Men's Performance 1/4 Zip Pullover Heather/Black | 1 |
| 8 pc Android Sticker Sheet | 1 |



**Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase**

*Purpose*: Evaluate drop-off rates at each step -> understand where customers drop in the conversion funnel, thereby evaluating the effectiveness of each step and prioritizing improving the experience at points with the highest drop-off rates.

```sql
with
product_view as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    count(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  and product.productRevenue is not null  
  group by 1
)

select
    pv.*,
    num_addtocart,
    num_purchase,
    round(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    round(num_purchase*100/num_product_view,2) as purchase_rate
from product_view pv
left join add_to_cart a on pv.month = a.month
left join purchase p on pv.month = p.month
order by pv.month;
```
**Result**
| month | num_product_view | num_addtocart | num_purchase | add_to_cart_rate | purchase_rate |
| --- | --- | --- | --- | --- | --- |
| 201701 | 25787 | 7342 | 2143 | 28.47 | 8.31 |
| 201702 | 21489 | 7360 | 2060 | 34.25 | 9.59 |
| 201703 | 23549 | 8782 | 2977 | 37.29 | 12.64 |

