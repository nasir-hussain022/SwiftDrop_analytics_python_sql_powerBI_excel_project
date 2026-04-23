# SwiftDrop Analytics: Architecting a Single Source of Truth for E-Commerce Profitability

## 📌 Project Overview
SwiftDrop Analytics is an end-to-end data pipeline and business intelligence solution designed to optimize the operations and profitability of a delivery-based e-commerce platform. This project bridges the gap between fragmented departments by synthesizing sales performance with operational logistics.

<img width="1126" height="628" alt="dashboard" src="https://github.com/user-attachments/assets/6e323223-53cb-486b-99eb-5c4e9319a0d1" />

## 🚀 Summary

### **Situation**
SwiftDrop, a growing e-commerce service, struggled to gain a clear view of its financial health and operational efficiency. With data spread across thousands of isolated order, product, and delivery records, the business faced unknown revenue leaks, high return rates, and inconsistent warehouse performance.

### **Task**
My goal was to build a robust analytics pipeline to answer critical business questions:
* What is the actual gross profit after accounting for discounts, COGS, and shipping?
* Which warehouses are underperforming in terms of delivery speed?
* How do discounts impact return rates?
* Which products are "low-margin" and threatening the bottom line?

### **Action**
I developed a multi-stage solution using **SQL, Python, and Power BI**:
* **Data Ingestion & Cleaning:** Used **MySQL** to ingest raw CSV data. Performed extensive cleaning, including standardizing date formats, trimming whitespace, and using **Regular Expressions (RegEx)** to extract measurement units (e.g., "kg", "ml") for feature engineering.
* **Advanced Analysis:** Utilized **Python (Pandas)** and **SQL CTEs** to calculate complex metrics such as Average Order Value (AOV) and SKU-level profitability.
* **Techniques:** Applied **Hypothesis Testing** to analyze the correlation between discounts and returns, and performed **Value Density Analysis** to identify high-profit, low-weight items.
* **Visualization:** Designed an interactive **Power BI Dashboard** to translate technical findings into visual trends for stakeholders.

### **Result**
* Established a **Single Source of Truth**, eliminating organizational information silos.
* Identified a **Total Net Revenue of $7.13M** and an **Absolute Gross Profit of $2.25M**.
* Pinpointed specific "Low-Margin" products (margin < 20%) for strategic repricing or removal.
* Empowered management to stop guessing and start making decisions based on warehouse efficiency and true product profitability.

---

## 🛠️ Technical Stack & Skills
* **Database:** MySQL (ETL, Data Cleaning, CTEs, Foreign Key Mapping)
* **Programming:** Python (Pandas, SQLAlchemy) for advanced data manipulation
* **Visualization:** Power BI (DAX, Interactive Dashboards)
* **Techniques:** Feature Engineering (RegEx), Hypothesis Testing, Value Density Analysis

## 📊 Business Questions Solved
1.  **Daily Revenue Trends:** Tracking growth day-over-day.
2.  **Warehouse Performance:** Identifying dispatch delays and speed bottlenecks.
3.  **Discount Strategy:** Analyzing if higher discounts lead to higher return rates.
4.  **Operational Health:** Measuring delivery time accuracy across regions.

## 📂 Data Sources
The project utilizes three core datasets:
* `orders.csv`: Sales transactions and discount data.
* `products.csv`: Inventory details, COGS, and retail pricing.
* `deliveries.csv`: Logistics data including dispatch and delivery timestamps.

---

## Database, table creation and insertion
**Database**
```sql
create database swiftdrop_analytics;
use swiftdrop_analytics;
```

**Table creation and data insertion**
```sql
-- Orders
create table orders (
order_id varchar(20),
customer_id	varchar(20), 
product_id varchar(20),	
order_date varchar(100),	
discount_applied float,
order_status varchar(20)
);

load data local infile "D:/NH projects/SwiftDrop/orders.csv"
into table orders
fields terminated by ','
lines terminated by '\n'
ignore 1 rows;

-- Products
create table products(
product_id	varchar(20),
category varchar(50),
product_name varchar(100),
cogs float,
retail_price float
);

load data local infile "D:/NH projects/SwiftDrop/products.csv"
into table products
fields terminated by ','
lines terminated by '\n'
ignore 1 rows;

-- Deliveries
create table deliveries(
delivery_id varchar(50),
order_id varchar(50),
warehouse_id varchar(50),
dispatch_time datetime,
delivery_time datetime,	
shipping_cost float
);

load data local infile "D:/NH projects/SwiftDrop/deliveries.csv"
into table deliveries
fields terminated by ','
lines terminated by '\n'
ignore 1 rows;

```

## Update Table
```sql
UPDATE orders 
SET 
    order_date = STR_TO_DATE(order_date, '%d-%m-%Y %H:%i');

UPDATE orders 
SET 
    order_status = TRIM(REPLACE(REPLACE(order_status, CHAR(13), ''),
            CHAR(10),
            ''));

alter table orders
modify column order_date datetime;

-- create 2 different columns
alter table products
add column measure_value float,
add column measure_unit varchar(20);


-- extract units from product_name and put values in measure_value and units in measure_unit

-- Update with logic for different categories
UPDATE products 
SET 
    measure_value = CASE
        WHEN
            REGEXP_SUBSTR(product_name, '[0-9]+', 1, 2) IS NOT NULL
        THEN
            CAST(CONCAT(REGEXP_SUBSTR(product_name, '[0-9]+', 1, 1),
                        '.',
                        REGEXP_SUBSTR(product_name, '[0-9]+', 1, 2))
                AS DECIMAL (10 , 2 ))
        ELSE CAST(REGEXP_SUBSTR(product_name, '[0-9]+', 1, 1)
            AS DECIMAL (10 , 2 ))
    END,
    measure_unit = CASE
        WHEN category = 'Groceries' THEN 'kg'
        WHEN category = 'Apparel' THEN 'pcs'
        WHEN
            category = 'Personal Care'
                AND product_name LIKE '%ml%'
        THEN
            'ml'
        WHEN product_name LIKE '%pack%' THEN 'pack'
        ELSE 'unit'
    END
WHERE
    product_id IS NOT NULL;

-- remove these units from the product_name column

UPDATE products 
SET 
    product_name = TRIM(REGEXP_REPLACE(REGEXP_REPLACE(product_name,
                        '[0-9]+(.[0-9]+)?s*[a-zA-Z]*',
                        ''),
                '[0-9]+$',
                ''));

```

**Add primary and foreign key**

```sql
-- Add foreign key and prinary key
alter table orders
add primary key(order_id);

alter table products 
add primary key(product_id);

alter table orders
add constraint products_fk_orders foreign key(product_id)
references products(product_id);

alter table deliveries
add constraint orders_fk_deliveries foreign key(order_id)
references orders(order_id);
```

## Analysis
**1. Total Net Revenue.**
```sql
SELECT 
    round(SUM(p.retail_price * (1 - o.discount_applied)),2) AS total_revenue
FROM
    products p
        INNER JOIN
    orders o ON o.product_id = p.product_id;
```
<img width="136" height="51" alt="image" src="https://github.com/user-attachments/assets/39b9fdd8-2af7-4562-a95c-3145b9f686ba" />

**2. Total Net Revenue by each category.**
```sql
SELECT 
    category, round(SUM(p.retail_price * (1 - o.discount_applied)),2) AS total_revenue
FROM
    products p
        INNER JOIN
    orders o ON o.product_id = p.product_id
    group by category;
```
<img width="215" height="112" alt="image" src="https://github.com/user-attachments/assets/99c66855-6674-4362-9134-36e402cbaf8f" />

**3. Absolute Gross Profit (profit after substracting the cost of goods and shipping cost).**
```sql
SELECT 
    CONCAT(ROUND(SUM((retail_price * (1 - discount_applied)) - cogs - shipping_cost) / 1000000,
                    2),
            'M') AS gross_profit
FROM
    products AS p
        LEFT JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    order_status = 'Delivered';
```
<img width="127" height="50" alt="image" src="https://github.com/user-attachments/assets/8a24415a-e77b-4b93-a6bc-fa91cac90028" />

**4. Profit Margin % per Category.**
```sql
SELECT 
    category,
    ROUND(SUM((retail_price * (1 - discount_applied)) - cogs - shipping_cost) / SUM(retail_price * (1 - discount_applied)) * 100,
            2) AS profit_margin
FROM
    products AS p
        LEFT JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    order_status = 'Delivered'
GROUP BY 1;
```
<img width="234" height="121" alt="image" src="https://github.com/user-attachments/assets/afaaca6c-0ce2-4139-91ee-4631cd91f40f" />

**5. Revenue Lost to Cancellations.**
```sql
SELECT 
    ROUND(SUM(retail_price * (1 - discount_applied)),
            2) AS revenue_lost
FROM
    products AS p
        LEFT JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    order_status = 'Cancelled';
```
<img width="156" height="49" alt="image" src="https://github.com/user-attachments/assets/373e100a-b8b9-48ea-9252-848bd4643d30" />

**6. Which product sold the most in each category.**
```sql
with product_sales as (
    select 
        p.category, 
        p.product_name,                     
        count(*) as product_sold
    from products p  -- <--- You were likely missing this line
    inner join orders o on o.product_id = p.product_id
    where o.order_status = 'Delivered'
    group by 1, 2
)
select category, product_name, product_sold
from (
    select 
        category, 
        product_name, 
        product_sold,
        row_number() over(partition by category order by product_sold desc) as rn
    from product_sales
) ranked
where rn = 1
order by product_sold desc;
```
<img width="314" height="120" alt="image" src="https://github.com/user-attachments/assets/fb295c4b-7038-4378-8167-781a50790387" />

**7. In which month product sold the most?**
```sql
  with product_sales as (
    select                                            
        p.category, 
        p.product_name, monthname(order_date) as month_name,
        count(*) as product_sold
    from products p  -- <--- You were likely missing this line
    inner join orders o on o.product_id = p.product_id
    where o.order_status = 'Delivered'
    group by 1, 2, 3
)
select category, product_name, product_sold, month_name
from (
    select 
        category, 
        product_name, 
        product_sold, month_name,
        row_number() over(partition by category order by product_sold desc) as rn
    from product_sales
) ranked
where rn = 1
order by product_sold desc;
```
<img width="391" height="122" alt="image" src="https://github.com/user-attachments/assets/4c106dd0-1dd9-42d0-a111-b9bc99ba2069" />

**8.Checks if orders with >15% discounts are resulting in a net loss after shipping.**
```sql
SELECT 
    ROUND(SUM((retail_price * (1 - discount_applied)) - cogs - shipping_cost),
            2) AS order_profit
FROM
    products AS p
        inner JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    discount_applied > 0.15
        AND order_status = "Delivered";
```
<img width="143" height="49" alt="image" src="https://github.com/user-attachments/assets/76e0bcde-a5a7-4d15-a022-59608e0bcafa" />

**9. Delivery Time (Order to Door).**
```sql
SELECT 
    order_date,
    delivery_time,
    CONCAT(FLOOR(TIMESTAMPDIFF(MINUTE,
                        order_date,
                        delivery_time) / 60),
            'h ',
            MOD(TIMESTAMPDIFF(MINUTE,
                    order_date,
                    delivery_time),
                60),
            'm') AS took_time_to_deliver
FROM
    orders AS o
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    YEAR(d.delivery_time) > 0;
```
<img width="388" height="124" alt="image" src="https://github.com/user-attachments/assets/0fe33244-31b0-4e12-a37c-e4fc6c3fd871" />

**10. Which warehouse is the fastest at getting packages out to the riders?**
```sql
SELECT 
    warehouse_id,
   CONCAT(FLOOR(TIMESTAMPDIFF(MINUTE,
                        order_date,
                        dispatch_time) / 60),
            'h ',
            MOD(TIMESTAMPDIFF(MINUTE,
                    order_date,
                    dispatch_time),
                60),
            'm') AS warehouse_get_packages
FROM
    orders o
        INNER JOIN
    deliveries d ON d.order_id = o.order_id
ORDER BY warehouse_get_packages ASC;
```
<img width="262" height="89" alt="image" src="https://github.com/user-attachments/assets/a1d2639c-0c0c-4734-8e27-25e3fd973eb8" />

**11. Late Delivery Detection. Lists orders that took longer than 4 hours (the threshold for "Quick Commerce").**
```sql

SELECT 
    warehouse_id,
    TIMESTAMPDIFF(MINUTE,
        order_date,
        delivery_time) / 60 AS dispatch_time
FROM
    orders o
        INNER JOIN
    deliveries d ON d.order_id = o.order_id
WHERE
    (TIMESTAMPDIFF(MINUTE,
        order_date,
        delivery_time) / 60) > 4;
        
```
<img width="205" height="122" alt="image" src="https://github.com/user-attachments/assets/d2d9ac3f-e434-493c-bb13-442de813986a" />

**12. Shipping Cost as % of Revenue. Are we spending too much on delivery relative to the product price?**
```sql
SELECT 
    category,
    ROUND(SUM(shipping_cost) / SUM(retail_price * (1 - discount_applied)) * 100,
            2) AS ship_cost_ratio
FROM
    products AS p
        INNER JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
GROUP BY category
ORDER BY ship_cost_ratio ASC;
```
<img width="211" height="117" alt="image" src="https://github.com/user-attachments/assets/c7430df8-6072-405b-a3ad-9ccac09d48aa" />

**13. Peak Hour Traffic.**
```sql
SELECT 
    DATE_FORMAT(order_date, '%h %p') AS order_hour,
    COUNT(*) AS total_order
FROM
    orders
GROUP BY 1
ORDER BY total_order DESC;
```
<img width="175" height="105" alt="image" src="https://github.com/user-attachments/assets/9c807e67-01e3-41e3-a3a7-cc1a14d5b454" />

**14. Top 5 Most Returned Products.**
```sql
SELECT 
    product_name, COUNT(*) AS products_returned
FROM
    products AS p
        INNER JOIN
    orders AS o ON o.product_id = p.product_id
WHERE
    order_status = 'Returned'
GROUP BY 1
ORDER BY products_returned DESC
LIMIT 5;
```
<img width="252" height="109" alt="image" src="https://github.com/user-attachments/assets/2425fe12-bede-49a1-9345-5799cb74d7de" />

**15. Average Order Value (AOV) by Customer.**
```sql
with order_sales as (SELECT 
    customer_id, order_id,
    sum(p.retail_price * (1 - o.discount_applied))
             AS order_total
FROM
    orders o
        JOIN
    products p ON o.product_id = p.product_id
GROUP BY 1, 2
)

SELECT 
    customer_id,
    ROUND(AVG(order_total),
            2) AS AOV
FROM
    order_sales
GROUP BY 1
ORDER BY AOV DESC;
```
<img width="170" height="92" alt="image" src="https://github.com/user-attachments/assets/218ac19c-7d0f-48bb-b9f9-478af762b0f7" />


**16. Most Profitable Individual Products.**
```sql
SELECT 
    product_name,
    ROUND(SUM(p.retail_price * (1 - o.discount_applied) - cogs - shipping_cost),
            2) AS total_revenue
FROM
    products p
        INNER JOIN
    orders o ON o.product_id = p.product_id
        INNER JOIN
    deliveries d ON d.order_id = o.order_id
WHERE
    order_status = 'Delivered'
GROUP BY 1
ORDER BY total_revenue DESC
LIMIT 1;

```
<img width="198" height="46" alt="image" src="https://github.com/user-attachments/assets/98d0cddb-0c37-4c39-b349-4f427fd7ed85" />

**17. Category Volume vs. Value. Does the category with the most orders also bring in the most money?**
```sql
SELECT 
    category,
    COUNT(order_id) AS order_count,
    ROUND(SUM(retail_price), 2) AS gross_value
FROM
    products AS p
        INNER JOIN
    orders AS o ON o.product_id = p.product_id
GROUP BY 1
ORDER BY category DESC , order_count DESC;
```
<img width="260" height="105" alt="image" src="https://github.com/user-attachments/assets/d4a4a912-cda4-44c0-b773-ecb2fd18fcce" />

**18. Weight vs Shipping Cost Correlation. Does the weight correlate with higher shipping fees?**
```sql
SELECT 
    CONCAT(measure_value, ' ', measure_unit) AS measure_unit,
    ROUND(AVG(shipping_cost), 2) AS avg_shipping_cost
FROM
    products AS p
        INNER JOIN
    orders AS o ON o.product_id = p.product_id
        INNER JOIN
    deliveries AS d ON d.order_id = o.order_id
WHERE
    category = 'Groceries'
GROUP BY 1;
```
<img width="212" height="90" alt="image" src="https://github.com/user-attachments/assets/d2cf1ae8-26e8-47d3-976f-4ef2bc82e92b" />

**19. Failed Delivery Rate per Warehouse.**
```sql
SELECT 
    warehouse_id,
    COUNT(CASE
        WHEN order_status != 'Delivered' THEN 1
    END) AS not_delivered
FROM
    orders o
        INNER JOIN
    deliveries d ON d.order_id = o.order_id
GROUP BY 1
ORDER BY not_delivered DESC;  
```
<img width="189" height="108" alt="image" src="https://github.com/user-attachments/assets/85ab318b-d153-4627-a6f9-18f6922e169c" />

**20. Lists products where the base profit margin is less than 20%.**
```sql

SELECT 
    product_name,
    ROUND(((retail_price - cogs) / retail_price) * 100,
            2) AS base_margin
FROM
    products
WHERE
    ((retail_price - cogs) / retail_price) < 0.20;
```
<img width="230" height="106" alt="image" src="https://github.com/user-attachments/assets/bc14b7d0-45d4-4cb6-a380-2bd6f9c88350" />

**21. Daily Revenue Trend. Tracks if the business is growing day-over-day.**
```sql
SELECT 
    DAY(order_date) AS days,
    ROUND(SUM(retail_price), 2) AS daily_revenue
FROM
    orders o
        INNER JOIN
    products p ON o.product_id = p.product_id
WHERE
    order_status = 'Delivered'
GROUP BY days
ORDER BY days ASC;
```
<img width="170" height="125" alt="image" src="https://github.com/user-attachments/assets/8f9e9536-c305-4aa0-a517-eec01ea2febf" />

**22. Discounts vs. Returns. Does offering higher discounts lead to higher return rates?**
```sql
SELECT 
    discount_applied, COUNT(*) AS return_count
FROM
    orders
WHERE
    order_status = 'Returned'
GROUP BY discount_applied
ORDER BY discount_applied DESC;
```
<img width="214" height="110" alt="image" src="https://github.com/user-attachments/assets/94d3fae7-f695-4007-b4d9-5533c77a24b0" />

**23. Value Density Analysis. Identifies high-value, low-weight items (the most profitable items to ship).**
```sql
SELECT 
    product_name,
    CONCAT(measure_value, ' ', measure_unit) AS weight,
   ROUND((retail_price / measure_value),2) AS value_density
FROM
    products
WHERE
    measure_unit = 'kg'
        OR measure_unit = 'Kg'
ORDER BY value_density DESC , weight ASC; 
```
<img width="251" height="124" alt="image" src="https://github.com/user-attachments/assets/a8360470-cea3-4975-b94f-0c787c1bfda8" />


---

## 👤 Author
**Nasir Hussain**
* Data Analyst
* [GitHub Profile](https://github.com/nasir-hussain022)
