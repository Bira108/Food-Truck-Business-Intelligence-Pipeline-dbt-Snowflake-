# Food Truck Business Intelligence Pipeline (dbt + Snowflake)

## Project Overview
This project establishes a robust, production-grade Analytics Engineering framework utilizing **dbt Cloud** and **Snowflake** to transform raw transactional data from the "Tasty Bytes" food truck dataset into clean, modeled, and tested business dimensions. The final data layer is fully optimized to feed analytical dashboards in Tableau Public.

## Data Architecture & Modeling Layers
I designed a synchronized, multi-layered data pipeline following industry-standard Kimball dimensional modeling principles to separate raw storage from business-facing data marts.

### 1. Staging Layer (`stg_`)
* **`stg_orders`**: Extracts raw order headers, standardizes naming conventions, implements defensive programming via `COALESCE` to eliminate missing customer profiles, and restricts processing to a rolling 30-day development window to minimize warehouse compute costs.
* **`stg_lineitems`**: Cleans individual transaction detail records, serving as the granular foundation for itemized sales tracking.

### 2. Marts Layer (`mart_`)
* **`mart_customer_orders`**: A granular fact table combining order summaries and line items to track item-level purchase behavior per transaction.
* **`mart_monthly_orders`**: A high-performance strategic summary mart that aggregates revenues, total items, and transaction frequencies by month and customer using `DATE_TRUNC`. This layer pre-computes heavy math to ensure near-instant dashboard loading times in Tableau.

### 3. Performance Optimization Layer (Incremental Loading)
* **`recent_orders`**: Implements an **Incremental Materialization Strategy** (`materialized='incremental'`). By utilizing the `is_incremental()` macro, this model dynamically scans the existing destination table for the maximum date and appends *only new data records* rather than rebuilding historical rows from scratch, drastically reducing Snowflake compute costs.

## Data Architecture & Pipeline Engineering
* **Data Warehouse Layer (Snowflake):** Raw, high-volume transactional logs from food truck point-of-sale (POS) systems were ingested and staged directly within a Snowflake data warehouse instance.
* **Transformation & Modeling Layer (dbt):** Developed modular SQL data transformations using dbt (Data Build Tool) to implement business logic.
    * Applied DATE_TRUNC and window functions to compute customer lifetime values and transactional aggregates.
    * Materialized clean dimension and fact models (mart_monthly_orders, mart_customer_orders) optimized for analytical query performance.
* **Business Intelligence Layer (Tableau):** Established a downstream analytical semantic layer. For local repository deployment and portability, enterprise dbt marts were extracted into a Tableau Packaged Workbook (.twbx) to display executive KPIs, revenue distributions, and user-segmentation insights seamlessly without requiring live warehouse credential exposure.

---

## Governance, Quality Control, and Automation

To transition this from a simple SQL script to an automated enterprise pipeline, I implemented automated testing and dependency management:

* **Dependency Resolution (`{{ ref() }}`)**: Eliminated hardcoded database schemas by utilizing dbt's relation mapping. This creates a clear **Directed Acyclic Graph (DAG)** and ensures tables automatically adjust between development and production environments.
* **Package Management (`packages.yml`)**: Integrated external macro libraries (`dbt-labs/dbt_utils`) into the project root directory to standardize surrogate key generations and analytical transformations.
* **Automated Schema Testing (`schema.yml`)**: Enforced structural integrity rules across all staging and core business marts. The pipeline automatically fails and flags anomalies if any critical column breaches data quality thresholds:
    * `unique`: Validates that primary keys contain absolutely no duplicate transactions.
    * `not_null`: Ensures zero "ghost sales" can enter down-stream dashboards by guaranteeing identity fields are populated.

## Business Insights & Analytics Narrative

A diagnostic review of our **Total Revenue Captured ($123.9M)** revealed a critical operational dependency: **96.92 %** of total revenue originates from completely anonymous walk-up checkouts (tracked via system-default Customer ID 0).

While this anonymous volume represents our baseline cash-flow stability, it leaves the business blind to individual retention patterns.

By utilizing dbt modeling to isolate and filter out this operational baseline, the **VIP Loyalty Performance** matrix reveals our top true individual consumer accounts. For instance, our highest-value registered loyalty members contribute significant repeat business (e.g., Customer Account keys generating upward of $3,000+ across high-frequency purchase milestones).

**Strategic Recommendation:** Marketing stakeholders should bypass generalized discounting and leverage these specific anonymized account keys to deploy targeted high-tier retention campaigns, maximizing the lifetime value of our known loyalty base while creating incentives to transition anonymous walk-ups into registered ecosystems.


