---
title: "Week 6 Worklog"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Establish a modern analytics engineering architecture using **dbt Core (`dbt-redshift`)** connected to Amazon Redshift Serverless.
* Build a **Staging layer (7 Views)** for cleaning and type casting, and a **Marts layer (3 Physical Tables)** modeled after the **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`).
* Deploy an automated data quality testing framework (**9 dbt tests**) enforcing uniqueness, non-nullability, and referential integrity.

### Tasks Completed This Week:
* **dbt Project Setup & Materialization Strategy:** Configured `dbt-redshift` credentials via environment variables (`profiles.yml` and `.env`); defined materialization rules in `dbt_project.yml` with **Staging** models materialized as `VIEW` (zero storage cost) and **Marts** materialized as physical `TABLE` (optimized for BI queries).
* **Engineered 7 Staging Views:** Mapped source tables in `models/staging/sources.yml` to the `raw_data` schema; built 7 Staging Views (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`) to normalize column names, cast types, handle `NULL`s, and fix native Olist spelling mistakes (`product_name_lenght` → `product_name_length`).
* **Developed Central Fact Table (`fct_orders`):** Authored `fct_orders.sql` at order grain, aggregating financial metrics (`total_item_value`, `total_freight_value`, `total_payment_value`), fulfillment logistics metrics (`delivery_duration_days`, `delay_days`, boolean flag `is_delayed`), and customer satisfaction metrics (`review_score`).
* **Developed Kimball Dimension Models:**
  * **`dim_customers`:** Resolved unique identity at `customer_unique_id`, computed cumulative **Customer Lifetime Value (`lifetime_spend`)**, order counts, and automated segmentation (`Returning Customer` vs. `One-Time Customer`).
  * **`dim_products`:** Joined English translation mappings to standardize product category names from Portuguese to English (`category_name`).
* **Automated Data Quality Testing:** Defined 9 automated tests in `models/marts/schema.yml` covering `unique`, `not_null`, and foreign key `relationships` (`fct_orders.customer_id` → `dim_customers.customer_id`); executed `dbt run` and `dbt test` achieving **`PASS=9 WARN=0 ERROR=0`** and generated the **dbt Data Lineage Graph**.

### Week 6 Achievements:
* Transformed 9 disparate raw tables into a pristine **Kimball Star Schema Data Warehouse** (`fct_orders`, `dim_customers`, `dim_products`) on Amazon Redshift Serverless.
* Mastered modular SQL engineering (CTEs + Jinja `{{ ref() }}`), automated dependency management, and generated end-to-end Data Lineage diagrams.
* Verified **100% data quality and referential integrity** across the analytical models via 9/9 passing automated dbt tests.

### References:
* <https://docs.getdbt.com/docs/core/connect-data-platform/redshift-setup>
* <https://docs.getdbt.com/best-practices/how-we-structure/2-staging>
* <https://docs.getdbt.com/best-practices/how-we-structure/4-marts>
* <https://docs.getdbt.com/docs/build/data-tests>
