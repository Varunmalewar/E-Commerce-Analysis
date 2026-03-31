# Olist E-commerce Data Analysis

End-to-end data cleaning and business intelligence analysis of the Olist Brazilian e-commerce platform. Covers 9 relational datasets, comprehensive data preprocessing, and 14 analytical requirements for Power BI dashboard visualization.

## Table of Contents

- [Project Overview](#project-overview)
- [Directory Structure](#directory-structure)
- [Datasets](#datasets)
- [Data Cleaning Pipeline](#data-cleaning-pipeline)
- [Client Requirements](#client-requirements)
- [Tools & Technologies](#tools--technologies)
- [How to Run](#how-to-run)

## Project Overview

This project analyzes e-commerce transaction data from Olist, Brazil's largest department store marketplace. The workflow spans from raw data extraction via SQL Server through Python-based cleaning to final export for Power BI dashboarding. The analysis covers customer behavior, product performance, seller metrics, fulfillment efficiency, and promotional simulations.

**Key Metrics:**
- 99,441 orders across 96,096 unique customers
- 32,951 products in 73 categories
- 3,095 sellers
- 1M+ geolocation records

## Directory Structure

```
Final lap/
├── README.md
├── client requirement.docx           # Client requirements document (14 analyses)
├── datasets/                         # Raw source data
│   ├── Olist.Data.Dictionary.2.pdf   # Data dictionary
│   ├── product_category_name_translation.csv
│   ├── olist_customers_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_products_dataset.csv
│   └── olist_sellers_dataset.csv
└── cleaned/                          # Cleaned outputs
    ├── Project2.1 (1).ipynb          # Jupyter notebook (full cleaning code)
    ├── Project2.1 (1).md             # Markdown export of notebook
    ├── cleaned_customers.xls
    ├── cleaned_geo.xls
    ├── cleaned_orders.xls
    ├── cleaned_olist_items.xls
    ├── cleaned_payments.xls
    ├── cleaned_products.xls
    ├── cleaned_reviews.xls
    └── cleaned_seller.xls
```

## Datasets

| Dataset | Rows | Columns | Description |
|---------|------|---------|-------------|
| Customers | 99,441 | 5 | Customer ID, unique ID, zip, city, state |
| Geolocation | 1,000,163 | 5 | Zip code lat/lng, city, state |
| Orders | 99,441 | 8 | Order lifecycle timestamps and status |
| Order Items | ~112K | 7 | Products per order, price, freight |
| Order Payments | 103,886 | 5 | Payment type, installments, value |
| Order Reviews | ~99K | 7 | Review scores and comments |
| Products | 32,951 | 9 | Category, dimensions, photos count |
| Sellers | 3,095 | 4 | Seller ID, zip, city, state |
| Category Translation | 71 | 2 | Portuguese to English category names |

### Entity Relationships

```
Customers ──1:1──> Orders ──1:N──> Order Items ──N:1──> Products
                                              ──N:1──> Sellers
                  Orders ──1:N──> Order Payments
                  Orders ──1:N──> Order Reviews
```

## Data Cleaning Pipeline

### 1. Customers Table
- Applied `str.title()` to city names
- Unicode normalization using `unidecode` (removed Portuguese accents)
- Removed non-alphabetic characters using regex `[^A-Za-z\s]`
- Result: 99,441 rows, 96,096 unique customer IDs, 3,345 repeat customers

### 2. Geolocation Table
- Removed duplicates: 1,000,163 → 738,009 rows
- Dropped `geolocation_lat` and `geolocation_lng` columns (city column sufficient for Power BI)
- Unicode normalization: unique cities reduced from 8,011 → 5,969
- Removed numbers and special characters from city names
- Removed apostrophes, asterisks, periods from city values
- Manual fix: standardized "Sao Joao Do Pau D Alho" variants
- Final aggressive deduplication: 738,009 → 19,612 rows

### 3. Products Table
- Dropped 6 unnecessary columns: `product_name_lenght`, `product_description_lenght`, `product_weight_g`, `product_length_cm`, `product_height_cm`, `product_width_cm`
- Left joined with category translation table to get English names
- Manually mapped 2 missing categories:
  - `pc_gamer` → `computers`
  - `portateis_casa_forno_e_cafe` → `small_appliances_home_oven_and_coffee`
- Dropped redundant join columns after merge
- Final columns: `product_id`, `product_category_name`, `product_photos_qty`

### 4. Sellers Table
- Applied `str.title()` to city names
- Removed non-alphabetic characters from city names
- Verified zero duplicates on `seller_id` and full row

### 5. Orders Table
- Filtered invalid timestamp sequences:
  - `order_approved_at` must be >= `order_purchase_timestamp`
  - `order_delivered_carrier_date` must be >= `order_approved_at`
- Reduced from 99,441 → 96,279 rows
- Removed 6 canceled orders with non-null delivery dates (data anomaly)
- Final status distribution: `delivered`, `shipped`, `canceled`

### 6. Payments Table
- Dropped `payment_sequential` and `payment_installments` columns
- Aggregated `payment_value` by `order_id` and `payment_type` using `sum()`
- Reduced from 103,886 → 101,686 rows

### 7. Reviews Table
- Cleaned and exported to `cleaned_reviews.xls`

### 8. Order Items Table
- Cleaned and exported to `cleaned_olist_items.xls`

## Client Requirements

The project delivers 14 analytical requirements for Power BI dashboard:

| # | Analysis | Description |
|---|----------|-------------|
| 1 | General KPIs | Total customers, orders, sales, avg order value, inactive customers (12 months) |
| 2 | Category Contribution | Percentage share with color coding: green (>5%), cream (3-5%), yellow (<3%) |
| 3 | Customer Churn | Year-over-year identification of customers who stopped purchasing |
| 4 | Photos vs Sales | Correlation between product photo count and sales quantity |
| 5 | Market Basket Analysis | Product pairs frequently purchased together in the same order |
| 6 | New Product Performance | Last 6 months category sales and orders for new launches |
| 7 | Seasonality | Monthly sales patterns to identify demand cycles |
| 8 | Price Change Impact | Average price change per category and effect on sales behavior |
| 9 | BOGO Simulation | "Buy One Get One Free" impact on profit margin for top 10 products |
| 10 | Payment Type Analysis | Sales distribution across payment methods |
| 11 | Delivery vs Reviews | Average review score: on-time vs late deliveries |
| 12 | Fulfillment Timing | Average hours per order stage (purchase → approval → carrier → delivery) |
| 13 | Category Maturity | First 90 days revenue vs most recent 90 days per category |
| 14 | Underperforming Sellers | High-rated sellers (>4.5) with <10% local (in-state) sales |

## Tools & Technologies

| Tool | Purpose |
|------|---------|
| Python 3.x | Core programming language |
| Pandas | Data manipulation and analysis |
| NumPy | Numerical operations |
| PyODBC | SQL Server database connectivity |
| Unidecode | Unicode text normalization |
| Regex (re) | Pattern matching and text cleaning |
| SQL Server | Data source (database: `campusx_project2`) |
| Jupyter Notebook | Interactive development environment |
| Power BI | Dashboard visualization (downstream) |
| Anaconda | Python distribution and package management |

### Key Python Libraries

```
pip install pandas numpy pyodbc unidecode openpyxl
```

## How to Run

1. **Prerequisites**
   - Python 3.x with Anaconda
   - SQL Server with ODBC Driver 17 installed
   - Database `campusx_project2` loaded with raw Olist datasets

2. **Configure Database Connection**
   Update the server name in the notebook:
   ```python
   server = r"YOUR_SERVER_NAME"
   database = "campusx_project2"
   ```

3. **Execute Notebook**
   Open `cleaned/Project2.1 (1).ipynb` in Jupyter Notebook and run all cells sequentially.

4. **Cleaned Output**
   8 cleaned `.xls` files will be generated in the `cleaned/` directory, ready for Power BI import.

## Data Dictionary Reference

See `datasets/Olist.Data.Dictionary.2.pdf` for complete field descriptions.

| Field Prefix | Table |
|-------------|-------|
| `customer_*` | Customers |
| `geolocation_*` | Geolocation |
| `order_*` | Orders |
| `order_item_*` | Order Items |
| `payment_*` | Payments |
| `review_*` | Reviews |
| `product_*` | Products |
| `seller_*` | Sellers |
