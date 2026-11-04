---
title: "Week 6 Worklog"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Implement a modern **Analytics Engineering** transformation architecture using **dbt Core (`dbt-redshift`)** connected to Amazon Redshift Serverless.
* Build the **Staging Layer (7 Views)** for data cleansing/standardization and the **Marts Layer (3 Physical Tables)** modeled as a **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`).
* Enforce automated data quality governance (**9 dbt data tests**) validating primary key uniqueness, non-nullability, and foreign key referential integrity.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :--- |
| **Mon (Day 2)** | - Initialize the **dbt Core (`dbt-redshift`)** project and configure environment-backed Redshift Serverless profiles (`profiles.yml` & `.env`) <br> - Configure materializations in `dbt_project.yml`: **Staging** models as lightweight `VIEW` objects and **Marts** models as physical `TABLE` objects | 02/11/2026 | 02/11/2026 | <https://docs.getdbt.com/docs/core/connect-data-platform/redshift-setup> |
| **Tue (Day 3)** | - Define raw source mappings in `models/staging/sources.yml` <br> - Develop **7 Staging Views** (`stg_orders`, `stg_order_items`, `stg_customers`, `stg_products`, `stg_payments`, `stg_reviews`, `stg_category_translation`): standardize column naming, enforce `CAST` types, handle `NULL` defaults, and fix Olist typos (`product_name_lenght` → `product_name_length`) | 03/11/2026 | 03/11/2026 | <https://docs.getdbt.com/best-practices/how-we-structure/2-staging> |
| **Wed (Day 4)** | - Engineer the core Fact table **`fct_orders` (`models/marts/fct_orders.sql`)** at order granularity: integrate financial metrics (`total_item_value`, `total_freight_value`, `total_payment_value`), delivery SLA metrics (`delivery_duration_days`, `delay_days`, `is_delayed`), and average `review_score` | 04/11/2026 | 04/11/2026 | *The Data Warehouse Toolkit (Kimball)* |
| **Thu (Day 5)** | - Engineer the 2 Dimension tables: <br>&emsp; + **`dim_customers`:** Resolve `customer_unique_id` grain, compute **Customer Lifetime Value (`lifetime_spend`)**, order frequency, and automated segmentation (`Returning Customer` vs. `One-Time Customer`) <br>&emsp; + **`dim_products`:** Join category translation mappings to standardize Portuguese product categories into English (`category_name`) | 05/11/2026 | 05/11/2026 | <https://docs.getdbt.com/best-practices/how-we-structure/4-marts> |
| **Fri (Day 6)** | - Author **9 automated data quality tests** in `models/marts/schema.yml` (`unique`, `not_null`, and foreign key `relationships` verifying `fct_orders.customer_id` → `dim_customers.customer_id`) <br> - Execute `dbt run` and `dbt test` achieving **`PASS=9 WARN=0 ERROR=0`** and generate the **dbt Data Lineage Graph** | 06/11/2026 | 06/11/2026 | <https://docs.getdbt.com/docs/build/data-tests> |

### Week 6 Achievements:
* Transformed 9 raw relational datasets into an enterprise-ready **Kimball Star Schema** (`fct_orders`, `dim_customers`, `dim_products`) in Amazon Redshift Serverless.
* Mastered modular SQL development using CTEs and Jinja `{{ ref() }}` macros with automated dependency resolution and **Data Lineage** documentation.
* Guaranteed **100% analytical data integrity** across all Fact and Dimension tables with 9/9 automated dbt tests passing.
