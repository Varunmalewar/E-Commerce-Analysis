# 🛒 Olist E-Commerce Analytics
### End-to-End Business Intelligence Analysis | Python · SQL Server · Power BI

> **Uncovering the story behind 100K+ transactions on Brazil's largest e-commerce marketplace — from raw data to boardroom-ready insights.**

## 🔗 Live Dashboard
👉 [View Interactive Power BI Dashboard](https://app.powerbi.com/view?r=eyJrIjoiZjFkZjc5YTEtOTg2OS00OTllLWJkMDYtMTAzNDU3MTZjMDY3IiwidCI6IjVlNWNiMWJhLTdmYjUtNDcyMi05OWVhLWZlZDg3MDRkOWFiYSJ9&pageName=d6e8881202963d2a1a16)

---

## 📌 Table of Contents
1. [Project Overview](#project-overview)
2. [Dataset Overview](#dataset-overview)
3. [Data Architecture](#data-architecture)
4. [Data Cleaning & Transformation](#data-cleaning--transformation)
5. [Power BI Dashboard](#power-bi-dashboard)
6. [Key Business Insights](#key-business-insights)
7. [Tools & Technologies](#tools--technologies)
8. [How to Run](#how-to-run)
9. [Directory Structure](#directory-structure)

---

## 🔍 Project Overview

Olist is Brazil's largest department store marketplace that connects small businesses to major retail channels. This project performs a comprehensive end-to-end analysis of Olist's transaction data spanning **September 2016 to October 2018** — covering the complete journey from raw SQL extraction and Python-based data cleaning to an interactive 5-page Power BI dashboard.

The analysis was driven by **14 client-defined business requirements** addressing four core pillars:

| Pillar | Focus |
|---|---|
| 📊 Executive Performance | Revenue, orders, payment methods, geographic distribution |
| 👥 Customer Behavior | Churn analysis, retention patterns, seller performance |
| 📦 Product Intelligence | Category rankings, market basket analysis, seasonality |
| 💰 Profitability | Pricing impact, BOGO simulation, new product performance |

---

## 📦 Dataset Overview

The dataset consists of **9 relational CSV files** sourced from SQL Server database `campusx_project2`:

| Dataset | Rows | Description |
|---|---|---|
| `olist_orders_dataset` | 99,441 | Order lifecycle with timestamps and status |
| `olist_order_items_dataset` | ~112,000 | Products per order, price, freight value |
| `olist_customers_dataset` | 99,441 | Customer ID, unique ID, location |
| `olist_products_dataset` | 32,951 | Category, dimensions, photo count |
| `olist_sellers_dataset` | 3,095 | Seller ID, location details |
| `olist_order_payments_dataset` | 103,886 | Payment type, installments, value |
| `olist_order_reviews_dataset` | ~99,000 | Review scores and comments |
| `olist_geolocation_dataset` | 1,000,163 | Zip code latitude/longitude |
| `product_category_name_translation` | 71 | Portuguese → English category mapping |

**Scale:**
- 🛍️ **99,441** total orders
- 👥 **96,096** unique customers
- 🏷️ **73** product categories
- 🏪 **3,095** sellers
- 📍 **1M+** geolocation records

---

## 🗄️ Data Architecture

```
Customers ──1:1──> Orders ──1:N──> Order Items ──N:1──> Products
                                              ──N:1──> Sellers
                   Orders ──1:N──> Payments
                   Orders ──1:N──> Reviews
                   Orders ──N:1──> Geolocation (via zip)
```

---

## 🔧 Data Cleaning & Transformation

All cleaning was performed in Python using Pandas. Raw data was extracted from SQL Server via `pyodbc` and exported as `.xls` files for Power BI consumption.

### 1. Customers Table
- Applied `str.title()` normalization to city names
- Unicode normalization using `unidecode` — removed Portuguese accents for consistency
- Stripped non-alphabetic characters using regex `[^A-Za-z\s]`
- **Result:** 99,441 rows · 96,096 unique customers · 3,345 repeat customers identified

### 2. Geolocation Table
- Removed duplicates: **1,000,163 → 738,009 rows**
- Dropped `geolocation_lat` and `geolocation_lng` (city-level granularity sufficient for Power BI)
- Unicode normalization reduced unique city count: **8,011 → 5,969**
- Removed numbers, special characters, apostrophes and asterisks from city names
- Manual standardization: fixed multi-variant spellings of `"Sao Joao Do Pau D Alho"`
- Final aggressive deduplication: **738,009 → 19,612 rows**

### 3. Products Table
- Dropped 6 non-essential columns: `product_name_lenght`, `product_description_lenght`, `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm`
- Left joined with category translation table for English category names
- Manually mapped 2 unmapped categories:
  - `pc_gamer` → `computers`
  - `portateis_casa_forno_e_cafe` → `small_appliances_home_oven_and_coffee`
- **Final columns retained:** `product_id`, `product_category_name`, `product_photos_qty`

### 4. Sellers Table
- Applied `str.title()` to city names
- Removed non-alphabetic characters
- Verified zero duplicate `seller_id` values

### 5. Orders Table
- Enforced logical timestamp sequence validation:
  - `order_approved_at >= order_purchase_timestamp`
  - `order_delivered_carrier_date >= order_approved_at`
- Removed 6 canceled orders with anomalous non-null delivery dates
- **Result:** 99,441 → **96,279 clean rows**

### 6. Payments Table
- Dropped `payment_sequential` and `payment_installments` columns
- Aggregated `payment_value` by `order_id` and `payment_type` using `sum()`
- **Result:** 103,886 → **101,686 rows**

### 7. Reviews & Order Items
- Cleaned and standardized — exported to `.xls` for Power BI

---

## 📊 Power BI Dashboard

The final dashboard spans **5 interactive pages** with 20+ DAX measures, cross-page navigation, and conditional formatting:

| Page | Description |
|---|---|
| 🏠 **Home** | Project introduction, problem statement, dataset overview, navigation |
| 📊 **Executive Overview** | KPI cards, revenue trend, payment split, category revenue, Brazil map |
| 👥 **Customer Analytics** | Churn by year, order frequency, review distribution, underperforming sellers |
| 📦 **Product Intelligence** | Category ranking, MBA matrix, seasonality, price distribution |
| 💰 **Profitability** | New product performance, BOGO simulation, revenue split |

**Key DAX techniques used:**
`AVERAGEX` · `RANKX` · `TREATAS` · `EXCEPT` · `TOPN` · `KEEPFILTERS` · `SUMX` · `DATEADD` · `DATEDIFF` · `ALLEXCEPT` · `SELECTEDVALUE`

---

## 💡 Key Business Insights

### 1. 🔴 Logistics Failure is the Primary Driver of Customer Churn

> Identified that **97% of Olist customers placed only 1 order**, indicating near-zero retention. Traced the root cause to an **average 262-hour (11-day) delivery time**, which correlated with a **1.4-point drop in review scores** for late deliveries (4.2 on-time vs 2.8 late). This suggests that logistics inefficiency — not product quality or pricing — is the primary churn driver, and that reducing delivery time is the single highest-leverage retention intervention available to the business.

**Data evidence:**
- 97% single-purchase customers
- On-time delivery avg review: **4.2 / 5.0**
- Late delivery avg review: **2.8 / 5.0**
- Carrier-to-customer stage = **76% of total fulfillment time**

---

### 2. 🟡 Revenue Concentration Risk & Untapped Cross-Sell Opportunity

> Discovered that **watches_gifts and health_beauty together contribute 18.3% of total platform revenue** despite Brazil's diverse 71-category marketplace — creating significant revenue concentration risk. Further analysis via Market Basket Analysis revealed that **computers + computers_accessories** had a Lift of **9.65** (bought together 9.65× more than by chance), signaling a high-confidence cross-sell opportunity. Bundling these categories or surfacing recommendation logic could reduce dependency on top 2 categories while growing mid-tier category revenue.

**Data evidence:**
- Top 2 categories = **18.3%** of total revenue
- Computers + accessories Lift: **9.65** (highest in dataset)
- baby + cool_stuff: **14 co-purchase pairs** (highest volume cross-sell)

---

### 3. 🟢 BOGO Promotion Viability Supported by Payment Concentration

> Simulated a **BOGO promotion on the top 10 products by order volume** and projected a **+R$869K revenue impact (+20.12% uplift)** — based on the assumption that doubling quantity sold with every second unit free results in 1.5× effective revenue. This finding is further supported by the discovery that **78.6% of all transactions relied solely on credit cards**, indicating that customers are already comfortable with higher-value transactions. Volume-based promotions tied to credit card payment flexibility could simultaneously improve revenue, payment diversification, and customer engagement.

**Data evidence:**
- BOGO simulated revenue uplift: **+R$869,000 (+20.12%)**
- Credit card transaction share: **78.6%**
- Boleto (cash alternative): only **17.2%**

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **Python 3.x** | Core data cleaning and transformation |
| **Pandas** | Data manipulation, merging, aggregation |
| **NumPy** | Numerical operations |
| **PyODBC** | SQL Server database connectivity |
| **Unidecode** | Unicode/accent normalization |
| **Regex (re)** | Pattern matching and text cleaning |
| **SQL Server** | Raw data source (`campusx_project2`) |
| **Jupyter Notebook** | Interactive development environment |
| **Power BI Desktop** | Dashboard and visualization layer |
| **DAX** | Business measures and KPI calculations |
| **Power Query (M)** | In-Power BI transformations |
| **Anaconda** | Python distribution and environment |

---

## ▶️ How to Run

### Prerequisites
- Python 3.x with Anaconda
- SQL Server with ODBC Driver 17
- Database `campusx_project2` loaded with raw Olist datasets

### Step 1 — Configure Database Connection
Open `cleaned/Project2.1 (1).ipynb` and update:
```python
server = r"YOUR_SERVER_NAME\SQLEXPRESS"
database = "campusx_project2"
```

### Step 2 — Install Dependencies
```bash
pip install pandas numpy pyodbc unidecode openpyxl
```

### Step 3 — Run Notebook
Open Jupyter Notebook and execute all cells sequentially. 8 cleaned `.xls` files will be generated in the `cleaned/` directory.

### Step 4 — Load into Power BI
Import the 8 cleaned `.xls` files into Power BI Desktop and connect relationships as per the data architecture above.

---

## 📁 Directory Structure

```
Final lap/
├── README.md
├── client requirement.docx
├── datasets/
│   ├── Olist.Data.Dictionary.2.pdf
│   ├── product_category_name_translation.csv
│   ├── olist_customers_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_products_dataset.csv
│   └── olist_sellers_dataset.csv
└── cleaned/
    ├── Project2.1 (1).ipynb
    ├── cleaned_customers.xls
    ├── cleaned_geo.xls
    ├── cleaned_orders.xls
    ├── cleaned_olist_items.xls
    ├── cleaned_payments.xls
    ├── cleaned_products.xls
    ├── cleaned_reviews.xls
    └── cleaned_seller.xls
```

---

## 📎 Data Dictionary Reference

See `datasets/Olist.Data.Dictionary.2.pdf` for complete field descriptions.

| Field Prefix | Table |
|---|---|
| `customer_*` | Customers |
| `geolocation_*` | Geolocation |
| `order_*` | Orders |
| `order_item_*` | Order Items |
| `payment_*` | Payments |
| `review_*` | Reviews |
| `product_*` | Products |
| `seller_*` | Sellers |

---

<div align="center">

**Built by Varun · S.B. Jain Institute of Technology, Management & Research · 2026**

*Olist public dataset · Power BI · Python · SQL Server*

</div>
