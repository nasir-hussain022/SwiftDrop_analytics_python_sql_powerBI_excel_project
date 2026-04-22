# SwiftDrop Analytics: Architecting a Single Source of Truth for E-Commerce Profitability

## 📌 Project Overview
SwiftDrop Analytics is an end-to-end data pipeline and business intelligence solution designed to optimize the operations and profitability of a delivery-based e-commerce platform. This project bridges the gap between fragmented departments by synthesizing sales performance with operational logistics.

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

## 👤 Author
**Nasir Hussain**
* Data Analyst
* [GitHub Profile](https://github.com/nasir-hussain022)
