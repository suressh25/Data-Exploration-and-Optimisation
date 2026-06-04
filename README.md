# Data Exploration and Optimisation

An end-to-end Databricks project covering exploratory data analysis, query benchmarking, and Delta table optimisation for an orders dataset.

## Table of Contents

- [Project Structure](#project-structure)
- [Data Sources](#data-sources)
- [Notebook Details](#notebook-details)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [License](#license)

## Project Structure

| Notebook | Purpose |
|----------|---------|
| `01_EDA_Orders` | Exploratory data analysis of the silver orders table |
| `02_Benchmark_Orders` | Baseline query benchmarking with physical plan inspection |
| `03_Optimize_Orders` | Delta layout optimisation with before/after benchmark comparison |

## Data Sources

| Table | Layer | Role |
|-------|-------|------|
| `databricks_cat.silver.orders_silver` | Silver | Primary dataset — used across all notebooks |
| `databricks_cat.gold.factorders` | Gold | Optional downstream context (EDA and benchmarking) |

### Key Columns

- `order_id`
- `customer_id`
- `product_id`
- `quantity`
- `total_amount`
- `order_date`
- `year`

## Notebook Details

### 01_EDA_Orders

Performs a comprehensive exploratory analysis of `orders_silver`:

- Table availability check and schema review
- Row count and dataset profile (date range, distinct customers, distinct products)
- Null and distinct value analysis across key columns
- Numeric distributions and descriptive statistics for `quantity` and `total_amount`
- Quartile and IQR-based outlier detection with lower/upper bounds
- Monthly revenue and order volume trends
- Top 15 products and customers by revenue
- Printed EDA findings summary

### 02_Benchmark_Orders

Establishes repeatable baseline query performance before any optimisation:

**Benchmark Queries:**
- `date_range_filter` — trailing 90-day scan
- `product_aggregation` — 180-day revenue and units by product
- `customer_level_aggregation` — 180-day revenue and order count by customer

**Metrics Captured:**
- Elapsed time
- Result rows
- File count
- Table size
- Partition columns
- Clustering columns
- Statistics state
- Partition filters
- Pushed filters
- Pruning hints

Uses `EXPLAIN` and `EXPLAIN FORMATTED` to inspect physical plans.

### 03_Optimize_Orders

Creates isolated benchmark copies of the silver table and compares three Delta layout strategies:

| Scenario | Table | Layout |
|----------|-------|--------|
| Managed base copy | `orders_silver_perf_base` | Managed Delta, no partitioning or clustering |
| Partitioned by year | `orders_silver_perf_year` | Managed Delta, `PARTITIONED BY (year)` |
| Liquid clustering | `orders_silver_perf_clustered` | `CLUSTER BY (order_date, customer_id, product_id)` then `CLUSTER BY AUTO` |

**Optimisation Steps Applied:**
- `OPTIMIZE ... ZORDER BY (customer_id, product_id, order_date)` on the base and partitioned copies
- `OPTIMIZE` (no Z-order) on the clustered copy — liquid clustering replaces Z-ordering
- A narrow column subset is cached for EDA convenience; cache is cleared before timed runs to measure layout improvements fairly

**Outputs:** Before/after elapsed time comparison per query and scenario, plus an improvement-vs-source percentage summary.

## Prerequisites

- Unity Catalog access to `databricks_cat.silver` and `databricks_cat.gold`
- Write permission on `databricks_cat.silver` to create the three benchmark copy tables
- Databricks Runtime with Delta Lake support (Photon-enabled or serverless SQL recommended for interactive performance workloads)

## Getting Started

Run the notebooks in order:

1. **01_EDA_Orders** — understand the data before making any changes
2. **02_Benchmark_Orders** — capture the baseline performance of the source table
3. **03_Optimize_Orders** — create optimised copies and compare results

Each notebook is self-contained and re-runnable. The optimisation notebook uses `CREATE OR REPLACE TABLE` so it is safe to re-execute.

## License

This project is open source and available under the MIT License.
