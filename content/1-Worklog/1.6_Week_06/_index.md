---
title: "Week 6: Kimball Star Schema Data Modeling & Data Quality Engineering with dbt Core"
weight: 6
---

### 🇬🇧 Week 06 Worklog

**1. Tasks Completed:**
- **dbt Core (`dbt-redshift`) Transformation Architecture Setup:**
  - Initialized the **dbt Core** project connected directly to **Amazon Redshift Serverless** via `profiles.yml` using environment-injected secrets (`DBT_REDSHIFT_PASSWORD`).
  - Configured `dbt_project.yml` to enforce a multi-layered materialization strategy: the **Staging** layer materialized as lightweight `VIEW` objects (zero additional storage overhead) and the **Marts** layer materialized as physical `TABLE` objects (optimized for analytical BI query performance).
- **Staging Layer Development (`public_staging` - 7 Views):**
  - Declared raw source mappings in `models/staging/sources.yml` pointing to the `raw_data` schema in Redshift.
  - Built 7 standardized staging models (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`):
    - Standardized column naming conventions, enforced explicit data type casting (`CAST`), and handled `NULL` defaults.
    - Fixed upstream Olist dataset typos (`product_name_lenght` $\rightarrow$ `product_name_length`, `product_description_lenght` $\rightarrow$ `product_description_length`) and pre-computed line-item totals (`total_item_value = price + freight_value`).
- **Kimball Star Schema Marts Layer (`public_marts` - 3 Physical Tables):**
  - **`fct_orders` (`models/marts/fct_orders.sql`):** Engineered the core order-granularity Fact table integrating:
    - *Financial Metrics:* `total_items`, `total_item_value`, `total_freight_value`, `total_payment_value`, `primary_payment_type`, and `max_installments`.
    - *Logistics & Delivery Performance:* Computed actual delivery duration (`delivery_duration_days`), estimated duration (`estimated_delivery_duration_days`), net delay days (`delay_days`), and a binary SLA breach flag (`is_delayed = 1` when actual delivery exceeds estimated date).
    - *Customer Satisfaction:* Aggregated order-level average `review_score`.
  - **`dim_customers` (`models/marts/dim_customers.sql`):** Engineered the Customer Dimension table resolving Olist's dual-identifier grain (`customer_id` per order vs. `customer_unique_id` per human customer). Calculated `first_order_date`, `most_recent_order_date`, `total_orders`, **Customer Lifetime Value (`lifetime_spend`)**, and automated behavioral segmentation (`Returning Customer` when `total_orders > 1` vs. `One-Time Customer`).
  - **`dim_products` (`models/marts/dim_products.sql`):** Engineered the Product Dimension table joining Portuguese category names with `stg_category_translation` to produce standardized English categories (`category_name`, defaulting to `'unknown'` on missing values) alongside physical dimensions and weight metrics.
- **Automated Data Quality Testing & Lineage Documentation:**
  - Authored 9 automated data quality assertions in `models/marts/schema.yml`:
    - Primary key `unique` and `not_null` constraints across `fct_orders.order_id`, `dim_customers.customer_id`, `dim_products.product_id`, and `dim_customers.customer_unique_id`.
    - Foreign key referential integrity (`relationships`) verifying that every `customer_id` in `fct_orders` maps to a valid record in `dim_customers`.
  - Executed `dbt run` and `dbt test` achieving **100% test pass rate (`PASS=9 WARN=0 ERROR=0`)** and generated the end-to-end **dbt Data Lineage Graph**.

**2. Key Deliverables & Results:**
- Transformed fragmented raw tables into a production-ready **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`) in Amazon Redshift Serverless.
- Applied modern **Analytics Engineering** practices: modular CTE SQL modeling with Jinja `{{ ref() }}` dependencies, version-controlled transformations, and automated schema testing.
- Validated 100% data integrity across all analytical marts with 9/9 automated dbt tests passing.

**3. References:**
- Kimball, R., & Ross, M.: *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling*.
- dbt Labs Documentation: Best Practices for Structuring dbt Projects (Staging vs. Marts) & Data Tests.
- dbt-redshift Adapter Reference: Materializations and Redshift Serverless Configuration.
