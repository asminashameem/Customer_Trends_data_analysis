# Customer_Trends_data_analysis
Data analysis using Python, SQL and Power BI

# 🛍️ Customer Shopping Behavior Analysis

**End-to-end analytics project using Python, MySQL, and Power BI**

> **Business question:** *How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?*

![Python](https://img.shields.io/badge/Python-pandas-blue)
![MySQL](https://img.shields.io/badge/SQL-MySQL-orange)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow)

---

## 📑 Table of Contents

1. [Project Overview](#1-project-overview)
2. [Business Problem](#2-business-problem)
3. [Objectives](#3-objectives)
4. [Tech Stack](#4-tech-stack)
5. [Dataset](#5-dataset)
6. [Project Workflow](#6-project-workflow)
7. [Data Preparation & Modeling (Python)](#7-data-preparation--modeling-python)
8. [Data Analysis (SQL)](#8-data-analysis-sql)
9. [Dashboard & Visualization (Power BI)](#9-dashboard--visualization-power-bi)
10. [Key Findings & Insights](#10-key-findings--insights)
11. [Business Recommendations](#11-business-recommendations)
12. [Limitations & Future Work](#12-limitations--future-work)
13. [Repository Structure](#13-repository-structure)
14. [How to Run This Project](#14-how-to-run-this-project)

---

## 1. Project Overview

A leading retail company wants to understand its customers' shopping behavior to improve **sales**, **customer satisfaction**, and **long-term loyalty**. Management has noticed changing purchasing patterns across demographics, product categories, and sales channels, and wants to know which factors (discounts, reviews, seasons, payment preferences) drive purchase decisions and repeat purchases.

This project analyzes a customer shopping behavior dataset of **3,900 records** end to end:

1. **Python** cleans and transforms the raw data, then loads it into MySQL.
2. **SQL (MySQL)** answers business questions on customer segments, loyalty, and purchase drivers.
3. **Power BI** presents the key patterns in an interactive dashboard.
4. This report summarizes the findings and recommendations.

---

## 2. Business Problem

Management has observed:

- Shifting purchasing patterns across **demographics** (age, gender, location).
- Different performance across **product categories**.
- Uncertainty about how **discounts, reviews, seasons, shipping, and payment preferences** affect purchases and loyalty.

**Core challenge:** convert raw shopping data into insights that guide marketing, product, and customer-engagement decisions.

---

## 3. Objectives

| # | Objective | Deliverable |
|---|-----------|-------------|
| 1 | Clean, validate, and transform the raw dataset | Python notebook |
| 2 | Store data in a structured database and query it for insights | MySQL + SQL script |
| 3 | Identify customer segments, loyalty patterns, and purchase drivers | SQL analysis |
| 4 | Build an interactive dashboard for stakeholders | Power BI (`.pbix`) |
| 5 | Communicate findings and recommendations | This report + presentation |
| 6 | Publish all work in a structured repository | This GitHub repo |

---

## 4. Tech Stack

| Layer | Tools |
|-------|-------|
| Data preparation | Python, pandas, Jupyter Notebook |
| Database connection | SQLAlchemy, PyMySQL |
| Database & analysis | MySQL |
| Visualization | Power BI Desktop |
| Version control | Git & GitHub |

---

## 5. Dataset

**File:** `customer_shopping_behavior.csv`
**Size:** 3,900 rows × 18 columns (19 columns after feature engineering, with one redundant column removed)

| Column | Description |
|--------|-------------|
| `customer_id` | Unique customer identifier |
| `age` | Customer age (18–70, mean ≈ 44) |
| `gender` | Male / Female |
| `item_purchased` | Product bought (25 unique items) |
| `category` | Product category (4 categories) |
| `purchase_amount` | Purchase value in USD (20–100, mean ≈ $59.76) |
| `location` | US state (50 locations) |
| `size` | Product size (4 sizes) |
| `color` | Product color (25 colors) |
| `season` | Season of purchase (4 seasons) |
| `review_rating` | Customer rating (2.5–5.0, mean ≈ 3.75) |
| `subscription_status` | Subscriber: Yes / No |
| `shipping_type` | Shipping option (6 types) |
| `discount_applied` | Whether a discount was applied: Yes / No |
| `promo_code_used` | Whether a promo code was used (removed, see §7) |
| `previous_purchases` | Number of earlier purchases |
| `payment_method` | Payment method used |
| `frequency_of_purchases` | How often the customer buys (e.g., Weekly, Monthly, Annually) |

### Quick dataset profile

| Metric | Value |
|--------|-------|
| Total records | 3,900 |
| Approx. total revenue | ≈ $233,000 (3,900 × $59.76 average) |
| Average purchase amount | $59.76 |
| Average review rating | 3.75 / 5 |
| Gender split | 68% male (2,652) · 32% female (1,248) |
| Largest category | Clothing: 1,737 records (≈ 44.5%) |
| Subscribers | ≈ 27% (1,053) · Non-subscribers ≈ 73% (2,847) |
| Purchases with a discount | ≈ 43% (1,677) · Without ≈ 57% (2,223) |
| Most common season | Spring (999 records) |
| Most common shipping type | Free Shipping (675 records) |

---

## 6. Project Workflow

```
Raw CSV (3,900 rows)
        │
        ▼
┌────────────────────────┐
│ 1. Python (pandas)     │  Inspect → clean → engineer features
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ 2. MySQL               │  Load via SQLAlchemy → run business queries
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ 3. Power BI            │  KPI cards, charts, slicers
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ 4. Report & Deck       │  Findings + recommendations
└────────────────────────┘
```

---

## 7. Data Preparation & Modeling (Python)

Notebook: `Customer_Shopping_Behavior_Analysis_python.ipynb`

### 7.1 Data inspection
- Loaded the CSV into pandas and reviewed structure with `head()`, `info()`, and `describe(include='all')`.
- Checked null values with `isnull().sum()`.

### 7.2 Data cleaning

| Issue | Action |
|-------|--------|
| **37 missing `Review Rating` values** (the only column with nulls) | Filled with the **median rating within each product category**, which respects differences between categories better than a single global median |
| Inconsistent column names | Converted to `snake_case` (lowercase, underscores) and renamed `purchase_amount_(usd)` → `purchase_amount` |
| **Redundant column** | Verified that `discount_applied` and `promo_code_used` are identical for every row (`(df['discount_applied'] == df['promo_code_used']).all()` → `True`), then dropped `promo_code_used` to avoid duplicate information |

```python
# Impute missing ratings with the median of each category
df['Review Rating'] = df.groupby('Category')['Review Rating'] \
                        .transform(lambda x: x.fillna(x.median()))

# Standardize column names
df.columns = df.columns.str.lower().str.replace(' ', '_')
df = df.rename(columns={'purchase_amount_(usd)': 'purchase_amount'})

# Drop the redundant column
df = df.drop('promo_code_used', axis=1)
```

### 7.3 Feature engineering

| New feature | How it was built | Purpose |
|-------------|------------------|---------|
| `age_group` | `pd.qcut(age, q=4)` → **Young Adult, Adult, Middle-aged, Senior** (equal-sized quartile groups) | Demographic segmentation |
| `purchase_frequency_days` | Mapped `frequency_of_purchases` to days: Weekly = 7, Fortnightly / Bi-Weekly = 14, Monthly = 30, Quarterly / Every 3 Months = 90, Annually = 365 | Numeric measure of buying frequency for loyalty analysis |

```python
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)

frequency_mapping = {
    'Fortnightly': 14, 'Weekly': 7, 'Monthly': 30,
    'Quarterly': 90, 'Bi-Weekly': 14, 'Annually': 365,
    'Every 3 Months': 90
}
df['purchase_frequency_days'] = df['frequency_of_purchases'].map(frequency_mapping)
```

### 7.4 Loading into MySQL

The cleaned DataFrame was written to a MySQL database using SQLAlchemy and PyMySQL, creating the `customer` table in `mydatabase`:

```python
from sqlalchemy import create_engine

engine = create_engine(
    f"mysql+pymysql://{username}:{password}@{host}:{port}/{database}"
)
df.to_sql("customer", engine, if_exists="replace", index=False)
```

> 🔒 **Security note:** keep database credentials out of the repository. Store them in environment variables or a `.env` file that is listed in `.gitignore`.

---

## 8. Data Analysis (SQL)

Script: `customer_behavior_sql.sql` (MySQL)

The cleaned data sits in a single `customer` table (one row per purchase record). Ten business questions were answered:

| # | Business question | SQL techniques |
|---|-------------------|----------------|
| 1 | Total revenue from **male vs. female** customers | `SUM`, `GROUP BY` |
| 2 | Which customers **used a discount but still spent more than the average** purchase amount? | Subquery in `WHERE` |
| 3 | **Top 5 products by average review rating** | `AVG`, `ROUND`, `ORDER BY`, `LIMIT` |
| 4 | Average purchase amount: **Standard vs. Express shipping** | `WHERE … IN`, `AVG` |
| 5 | Do **subscribers spend more**? Compare customers, average spend, and total revenue | Multiple aggregates |
| 6 | **Top 5 products with the highest share of discounted purchases** | `CASE WHEN`, percentage calculation |
| 7 | **Customer segmentation** into New, Returning, Loyal by previous purchases | `CASE WHEN` segmentation |
| 8 | **Top 3 most purchased products within each category** | CTE + `ROW_NUMBER() OVER (PARTITION BY …)` |
| 9 | Are **repeat buyers (more than 5 previous purchases)** also likely to subscribe? | Filtered aggregation |
| 10 | **Revenue contribution of each age group** | `SUM`, `GROUP BY`, `ORDER BY` |

### Sample queries

**Subscribers vs. non-subscribers**
```sql
SELECT subscription_status,
       COUNT(customer_id)         AS total_customers,
       ROUND(AVG(purchase_amount),2) AS average_spend,
       SUM(purchase_amount)       AS total_revenue
FROM customer
GROUP BY subscription_status;
```

**Loyalty segmentation**
```sql
SELECT CASE WHEN previous_purchases < 2 THEN 'New'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal' END AS customer_type,
       COUNT(*) AS number_of_customer
FROM customer
GROUP BY customer_type;
```

**Top 3 products per category**
```sql
WITH item_count AS (
    SELECT category, item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category
                              ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_count
WHERE item_rank <= 3;
```

Segment definitions used in the analysis:

| Segment | Rule (`previous_purchases`) |
|---------|-----------------------------|
| New | fewer than 2 |
| Returning | 2 to 10 |
| Loyal | more than 10 |

---

## 9. Dashboard & Visualization (Power BI)

File: `customer_behavior_dashboard.pbix`, titled **"Customer Behavior Dashboard"**. It is a single-page, 1920×1080 interactive report connected to the `customer` table.

### 9.1 KPI cards
- **Number of Customers**
- **Average Purchase Amount**
- **Average Review Rating**

### 9.2 Visuals

| Visual | Type | What it shows |
|--------|------|---------------|
| Percentage of customers by subscription status | Donut chart | Subscriber vs. non-subscriber share |
| Revenue by category | Clustered column chart | Total purchase amount per product category |
| Sales by category | Clustered column chart | Number of customers per category |
| Revenue by age group | Clustered bar chart | Total purchase amount per age group |
| Sales by age group | Clustered bar chart | Number of customers per age group |

### 9.3 Interactive filters (slicers)
Users can filter the whole page by **Subscription status**, **Gender**, **Category**, and **Shipping type**.

### 9.4 Dashboard preview

> <img width="1182" height="670" alt="image" src="https://github.com/user-attachments/assets/8eac8cf4-4214-440f-9035-17779c5aab21" />

> `![Customer Behavior Dashboard](dashboard/dashboard_preview.png)`

---

## 10. Key Findings & Insights

> ⚠️ **Fill in the bracketed values from your SQL output and dashboard.** Everything else in this section comes directly from your notebook's exploratory output.

### 10.1 Customer base (from EDA)
- The data covers **3,900 purchase records** across **50 US locations** and **25 products** in **4 categories**.
- Customers are aged **18–70** (mean ≈ 44). About **68% are male**.
- Only **≈ 27% are subscribers**, leaving a large non-subscriber base.
- About **43% of purchases used a discount** (and every discounted purchase also used a promo code).
- The typical purchase is about **$60**, and the average rating is **3.75 / 5**, which is moderate satisfaction with room to improve.
- **Clothing** is the biggest category at ≈ 44.5% of records.

### 10.2 SQL results

| Question | Result |
|----------|--------|
| Revenue: male vs. female | Male: **157890** · Female: **75191** |
| Discount users spending above average | **200** customers |
| Top-rated products | **Gloves, Sandals, Boots, Hat, Handbag** (ratings **3.86**–**3.84**) |
| Avg. spend: Express vs. Standard | Express **60.48** · Standard **58.46** |
| Subscribers vs. non-subscribers | Avg. spend **59.49** vs. **59.87**; revenue **62645** vs. **170436** |
| Products most often bought with a discount | **Hat, Snikers,  Coat, Sweater, Pants** (up to **50%**) |
| Loyalty segments | New **83** · Returning **701** · Loyal **3116** |
| Top products per category | Clothing: **Blouse** · Footwear: **Sandals** · Accessories: **Jwellery** · Outerwear: **Jacket** |
| Repeat buyers who subscribe | Subscribed **958** vs. not subscribed **2518** |
| Revenue by age group | Young Adult **62143** · Adult **55978** · Middle-aged **59197** · Senior **55763** |

### 10.3 Dashboard takeaways
- **Highest-revenue category:** **[Clothing]** (**[44.73]%** of revenue).
- **Highest-revenue age group:** **[Young Adult]** (**[26.66]%** of revenue).
- **Subscription mix:** confirm the donut chart matches the ≈ 27% / 73% split.

---

## 11. Business Recommendations

Each recommendation should be backed by the matching result in §10. The evidence column shows where to look.

| # | Area | Recommendation | Evidence |
|---|------|----------------|----------|
| 1 | **Subscription growth** | Only about a quarter of customers subscribe. Promote subscription benefits to repeat buyers and high-frequency shoppers, especially if repeat buyers show low subscription rates. | §10.1, SQL Q5 & Q9 |
| 2 | **Targeted marketing** | Concentrate campaigns on the age groups and category that generate the most revenue, and tailor messaging by gender since the base is skewed male. | §10.3, SQL Q1 & Q10 |
| 3 | **Smarter discounting** | Discounts feature in ~43% of purchases. Focus offers on products with low discount rates or weak sales, and avoid blanket discounts on products that already sell well. | SQL Q2 & Q6 |
| 4 | **Loyalty programme** | Use the New / Returning / Loyal segments to design rewards, such as welcome offers for New customers and early access or points for Loyal customers. | SQL Q7 |
| 5 | **Product strategy** | Promote each category's top products, feature top-rated items, and investigate low-rated ones. Average rating of 3.75 suggests room to improve. | SQL Q3 & Q8 |
| 6 | **Shipping strategy** | Compare Express vs. Standard spending and consider shipping incentives (e.g., free-shipping thresholds) that raise order value. | SQL Q4 |
| 7 | **Seasonal planning** | Plan inventory and campaigns around peak seasons (Spring has the most records). | §5 profile |

---

## 12. Limitations & Future Work

**Limitations**
- The dataset is a snapshot with **no purchase dates or transaction timestamps**, so time-based trends can't be measured.
- The dataset has **no online vs. offline channel field**, so channel comparison was not possible.
- Each row is one purchase record per customer, so true repeat-purchase behavior is only inferred from `previous_purchases` and purchase frequency.
- There is **no cost or margin data**, so profitability wasn't assessed.
- Findings show association, not causation.

**Future work**
- Build **RFM segmentation** and clustering for richer customer groups.
- Develop a **predictive model** for subscription or repeat-purchase likelihood.
- Analyze **season and payment method** more deeply in SQL and Power BI (the data is already prepared).
- Add more dashboard pages (loyalty, payment and shipping, location map).
- Simulate transaction dates and channels to enable trend and channel analysis.

---

## 13. Repository Structure

```
customer-shopping-behavior-analysis/
│
├── data/
│   └── customer_shopping_behavior.csv
│
├── python/
│   └── Customer_Shopping_Behavior_Analysis_python.ipynb
│
├── sql/
│   └── customer_behavior_sql.sql
│
├── dashboard/
│   ├── customer_behavior_dashboard.pbix
│   └── dashboard_preview.png
│
├── docs/
│   ├── Business_Problem__Document.pdf
│   └── presentation.pptx
│
├── .gitignore
└── README.md
```


---

## 14. How to Run This Project

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
2. **Install dependencies**
   ```bash
   pip install pandas sqlalchemy pymysql jupyter
   ```
3. **Prepare the data:** open `python/Customer_Shopping_Behavior_Analysis_python.ipynb`, set the CSV path, and run all cells.
4. **Set up MySQL:** create the database (`CREATE DATABASE mydatabase;`), then update the connection details in the notebook's last cell (use environment variables for the password) and run it to create the `customer` table.
5. **Run the SQL analysis:** open `sql/customer_behavior_sql.sql` in MySQL Workbench and execute the queries.
6. **Open the dashboard:** open `dashboard/customer_behavior_dashboard.pbix` in Power BI Desktop and refresh the data source if needed.

---

