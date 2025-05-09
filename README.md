# E-Commerce-Sales-Customer-Insights-Dashboard

## Project Overview

E-commerce businesses generate large volumes of transactional data daily, but many struggle to extract actionable insights from it. This project aims to develop a data-driven, interactive Tableau dashboard that provides a comprehensive overview of sales, customer behavior, and inventory trends.

## Data Description

The project is based on the Brazilian E-Commerce Public Dataset by Olist, a multi-table dataset that captures detailed information about the e-commerce order lifecycle from 2016 to 2018. It includes data on customer behavior, product performance, sales transactions, delivery performance, and geolocation details.

The dataset comprises the following key tables:

- orders : Contains order-level information including timestamps for order placement, approval, shipping, and delivery.
- customers : Details of customers, including unique IDs and location metadata.
- order_items : Line-item details for each order, including product ID, seller ID, price, and freight charges.
- products : Contains product information such as category (original and translated), dimensions, and weight.
- sellers : Metadata about sellers and their location.
- order_payments : Payment details including type and number of installments.
- order_reviews : Customer feedback including review score and comment.
- geolocation : Contains zip code-level geolocation information including city, state, latitude, and longitude.
- product_category_name_translation : Maps Portuguese product category names to English.

## Methodology

- Data Cleaning
- Analytical Approach
- Findings
- Recommendations
- Conclusion

## Data Cleaning

The first step I took was to create a staging dataset for each table. I then standardized the data types of each column to ensure consistency and accuracy across the dataset. After this, I performed an exploratory analysis to understand the structure and contents of the data. I discovered that the customer_id field represented each individual transaction, while the unique_customer_id referred to each distinct customer. The presence of duplicate customer_id values was intentional and indicated multiple purchases made by the same customer.

Next, I analyzed the relationship between customer ZIP codes and geolocation ZIP codes to identify potential similarities. To do this, I used the following code:

```sql
SELECT
c.customer_id,
c.customer_zip_code_prefix,
g.geolocation_zip_code_prefix
FROM
customers_staging c
JOIN
geolocation_staging g ON c.customer_zip_code_prefix = g.geolocation_zip_code_prefix
WHERE
c.customer_zip_code_prefix <> g.geolocation_zip_code_prefix
LIMIT 1000;
```
The result of the ZIP code comparison showed zero matches, confirming that the customer_zip_code_prefix refers to the customer's location, while the geolocation_zip_code_prefix likely corresponds to the store or warehouse location.

During the data cleaning process, I also noticed that the geolocation_city column contained numerous inconsistencies and formatting issues, resulting in 5,959 unique (and often incorrect) city names. To resolve this, I used Python to access the OpenStreetMap API and retrieve a list of correctly formatted city names. This standardized list was then used to update and replace the inaccurate entries in the geolocation_city column.

```python
import mysql.connector
import requests
import time

# 🔹 Connect to MySQL
conn = mysql.connector.connect(
    host="localhost",
    user="****",
    password="*****",
    database="*****"
)
cursor = conn.cursor()

# 🔹 Create `temp_city_updates` table if it doesn't exist
cursor.execute("""
    CREATE TABLE IF NOT EXISTS temp_city_updates (
        incorrect_city VARCHAR(255) NOT NULL,
        correct_city VARCHAR(255) NOT NULL,
        PRIMARY KEY (incorrect_city)
    )
""")
conn.commit()

def get_correct_city(city_name):
    """Fetches corrected city name from cache or OpenStreetMap API"""
```
For the order_times data, I confirmed that there were no duplicate entries. The only transformation needed was formatting the datetime column to a date-only format to support more effective time-based analysis.

While cleaning the orders table, I identified that several values were missing in key datetime columns: order_delivered_carrier_date, order_approved_at, and order_delivered_customer_date. To better understand the pattern and extent of these missing values, I used a specific query to investigate further:

```sql
SELECT order_status,
COUNT(*) AS missing_date_count
FROM orders_staging
WHERE order_delivered_carrier_date IS NULL OR order_delivered_carrier_date = ''
GROUP BY order_status
ORDER BY missing_date_count DESC;
```
![dc sql 1](https://github.com/user-attachments/assets/57d7c6fd-5f81-4428-b139-d311b195f2a6)

![dc sql 2](https://github.com/user-attachments/assets/c1ba1f61-4df2-4e08-ba0e-d01eb01522f8)

![dc sql 3](https://github.com/user-attachments/assets/5c5aa140-df18-474a-a685-29502bb854bf)

## Handling Incomplete and Invalid Order Data
- Canceled Orders: All orders with a status of “canceled” were archived for future analysis rather than deleted, as they may offer insights into customer behavior or fulfillment issues.
- Unavailable Orders: Orders with corrupt or invalid entries were removed entirely from the dataset to prevent analytical errors.
- Missing Values in Key Columns:
Rows with missing values in critical columns such as order_delivered_customer_date and order_delivered_carrier_date were archived if they had a "delivered" status. This decision was made because missing delivery dates for completed orders suggest potential data integrity issues.
Orders that were marked as “approved” but had missing approval dates were also archived for further investigation.
- Created but Not Placed Orders: Orders with a status of “created” but no subsequent progress (no approval, delivery, or payment data) were deleted, as they represented incomplete transactions.

## Cleaning Duplicate and Redundant Data
Next, I addressed duplicate and irrelevant data:

I removed unnecessary tables and columns that were not useful for the scope of my analysis. These included:
- product_name_lenght
- product_description_lenght
- product_photos_qty
- product_weight_g
- product_length_cm
- product_height_cm
- product_width_cm
Upon inspection, the product_id column contained no duplicates, so no action was needed there.

## Translating Product Categories to English
- Since the product categories were originally listed in Portuguese, I created a mapping table containing the Portuguese-to-English translations.
- I joined this mapping table with the products table using the product category name as the key.
- After confirming accuracy, I updated the product_staging table with the English category names for consistency and better accessibility in analysis.

## Final Preparation for Visualization
- I performed the necessary joins between all cleaned and prepared tables to establish the correct relationships for analysis.
- The resulting dataset was structured and optimized for use in Tableau, enabling effective visualization and dashboard creation.

![Sales Performance Overview](https://github.com/user-attachments/assets/5fec7e7a-1a81-4d59-a9f2-a9966ef96e8f)

![Customer Insight ](https://github.com/user-attachments/assets/e1f693c6-0eda-4ae5-aa04-76f647289a99)

![Location and product](https://github.com/user-attachments/assets/e5c9f57f-8c59-45e8-9f8a-545fa6dd76fc)

## Analytical Approach
1. Data Understanding & Collection
- Imported and explored the Brazilian E-Commerce Public Dataset (Olist).
- Understood key tables such as orders, customers, order_items, products, sellers, payments, geolocation, and category translations.
- Assessed how each dataset contributed to different aspects of the business (sales, customer behavior, inventory, geography).
2. Data Cleaning & Preprocessing
- Handled missing values (e.g., incomplete delivery dates).
- Removed or archived invalid records (e.g., archived = 1).
- Standardized product category names using the product_category_name_translation table.
- Merged datasets to create master tables for focused analysis (e.g., sales performance, customer behavior).
- Removed duplicates and normalized formats (especially for date fields and product names).
3. Data Integration
- Joined relevant tables to create analytical views:
- Orders + Customers + Geolocation for customer and geographic analysis.
- Orders + Order Items + Products for sales and inventory analysis.
- Orders + Payments for revenue tracking.
4. Exploratory Data Analysis (EDA)
- Performed descriptive analysis using SQL and Tableau to identify trends and anomalies.
- Created summary statistics on revenue, order count, customer distribution, and product performance.
- Analyzed purchase patterns over time (monthly, yearly) and across customer segments.
5. KPI Definition & Dashboarding
- Defined KPIs such as:
- Total Sales
- Average Order Value (AOV)
- Total Orders
- Customer Lifetime Value (CLV)
- Built dashboards in Tableau using visual elements like:
- KPI cards
- Line charts (monthly trends)
- Bar charts (top/bottom performers)
- Pie charts (customer segments)
- Heatmaps (spending behavior)
- Maps (geographic distribution)
6. Segmentation & Advanced Insights
- Segmented customers into Loyal, At-Risk, and Lost based on purchase recency.
- Identified high-CLV customers and ranked product performance.
- Conducted comparative year-over-year (YoY) analysis.

## Findings
### Sales Performance
- Steady Revenue Growth: Monthly sales showed a consistent upward trend from 2016 to 2018, with peaks during national holidays and the end-of-year shopping season.
- Top Revenue Categories: Categories like health beauty, watches gifts, and computers accessories contributed the highest to total sales.
- Average Order Value (AOV): The AOV remained stable across years, with noticeable increases during promotion-heavy periods.
- Order Volume Insights: A small group of repeat customers generated a disproportionately high number of orders, indicating potential for loyalty programs.
### Customer Segmentation & Behavior
- Customer Loyalty Distribution:
Loyal: 22% of customers made purchases consistently across multiple months.
- At-Risk: 18% showed declining activity over time.
- Lost: 60% of customers did not return after a single purchase.
- Customer Lifetime Value (CLV): A small cohort of customers (Top 50) contributed significantly more to revenue than the rest, suggesting the Pareto Principle (80/20 Rule) is applicable.
- Purchase Frequency: Most customers made only one purchase, highlighting an opportunity to improve retention.
### Inventory & Product Performance
- Best-Selling Products: Items in the auto, bed_bath_table and sports_leisure categories consistently topped sales volume.
- Low Performing Products: Niche categories such as security and services and fashion kids had minimal contribution to sales, suggesting potential for stock optimization or discontinuation.
- Price vs Sales Volume: A weak correlation indicated that lower-priced items sold in high volume, while premium items generated more revenue per unit.
### Geographic Sales Insights
- Top Performing States: São Paulo (SP), Rio de Janeiro (RJ), and Minas Gerais (MG) were the highest in terms of order volume and revenue.
- Regional Distribution: Southern and Southeastern regions outperformed others significantly.
- Delivery Times: Orders to remote northern regions faced longer delivery times, which may impact customer satisfaction

## Recommendations
1. Sales & Revenue Trends
- Insights:

Sales peaked during [peak months] and declined during [slow months].
Year-over-year analysis shows [growth/decline/stability] in total revenue.
- Recommendations:

Increase marketing spend and stock inventory leading into peak months.
Launch promotional campaigns or discounts during slow periods to boost revenue.

2. Product Performance
- Insights:

Top-selling products like [Product A, Product B] contribute significantly to overall sales.
Least-selling products showed consistently low sales volume and low revenue contribution.
- Recommendations:

Focus marketing campaigns around top-selling products.
Consider discounting, bundling, or discontinuing poor-performing products to optimize inventory and cash flow.

3. Customer Segmentation & Behavior
- Insights:

Majority of customers are classified as [At-Risk/Lost], with few categorized as Loyal.
Top 50 customers generate a disproportionately large share of total sales.
- Recommendations:

Implement a customer loyalty program to retain and nurture existing customers.
Target at-risk and lost customers with win-back email campaigns or exclusive offers.
Create a VIP experience for top customers (early access to sales, premium support).

4. Geographic Sales Insights
- Insights:

States like [State A, State B] drive the majority of sales.
Other regions show potential but underperform in sales volume and revenue.
- Recommendations:

Target high-performing states with localized promotions and exclusive regional offers.
Analyze shipping, pricing, or advertising gaps in underperforming areas and adjust strategies accordingly.

5. Pricing and Sales Volume Relationship
- Insights:

Mid-priced products tend to sell at higher volumes than premium-priced products.
- Recommendations:

Focus pricing strategies around the sweet spot where customers show maximum purchasing intent.
For higher-end products, enhance perceived value through better marketing, bundling, or premium branding.

## Conclusion
This project provided a comprehensive analysis of the Olist e-commerce dataset, focusing on key business areas such as sales performance, customer behavior, product performance, and geographic sales distribution. Through careful data cleaning, integration, and visualization, we uncovered insights into revenue trends, top-performing products, and customer segmentation. The final dashboards offer actionable intelligence that can support strategic decisions in marketing, inventory management, and customer retention. Overall, the analysis highlights the power of data-driven storytelling in understanding and optimizing e-commerce operations.





