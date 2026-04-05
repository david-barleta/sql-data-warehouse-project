# Data Dictionary: Semantic Model Layer
This document covers all tables and columns in the semantic model (gold) layer of this project. The gold layer is built on top of the silver layer and serves as the business-ready layer, structured as a star schema, to support business intelligence & analytics.

**Tables covered:**
1. `gold.fact_sales`
2. `gold.dim_customers`
3. `gold.dim_products`

**Naming conventions:** Tables follow a `fact_` / `dim_` prefix convention. Column names use snake_case. Surrogate keys use the `_key` suffix. Business identifiers from the source system uses the `_id` or `_number` suffixes.

**Data types used:**

| Type           | Description                                       |
| -------------- | ------------------------------------------------- |
| `INT`          | Whole number                                      |
| `DECIMAL`      | Decimal number                                    |
| `NVARCHAR(50)` | Variable-length Unicode text, up to 50 characters |
| `DATE`         | Date value only, no time component (YYYY-MM-DD)   |

---

## Fact Tables
Fact tables store measurable, transactional events. Each row represents a business event at a specific level of granularity. Fact tables link to dimension tables via surrogate keys.

### `gold.fact_sales`
**Description:** Stores transactional sales data at the order line item level, with some orders containing multiple products (multiple rows per order). Contains surrogate keys linking to the customer and product dimensions, order lifecycle dates, and financial measures. Sales amount is validated against quantity and price using business rules applied in the silver layer.

**Granularity:** One row per sales order line item.

**Source:** `silver.crm_sales`, joined with `gold.dim_customers` and `gold.dim_products` for surrogate key lookup.

| Column        | Data Type    | Key                  | Description                                                                                                                               |
| ------------- | ------------ | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| order_number  | NVARCHAR(50) |                      | Alphanumeric identifier for each sales order (e.g., "SO54496"). Not unique in this table since one order can contain multiple line items. |
| product_key   | INT          | FK → `dim_products`  | Surrogate key linking the order line item to the product dimension.                                                                       |
| customer_key  | INT          | FK → `dim_customers` | Surrogate key linking the order line item to the customer dimension.                                                                      |
| order_date    | DATE         |                      | The date the order was placed.                                                                                                            |
| shipping_date | DATE         |                      | The date the order was shipped to the customer. Always on or after `order_date`.                                                          |
| due_date      | DATE         |                      | The date the order payment was due.                                                                                                       |
| sales_amount  | DECIMAL      |                      | Total monetary value of the line item in currency units.                                                                                  |
| quantity      | INT          |                      | Number of units ordered for this line item.                                                                                               |
| price         | DECIMAL      |                      | Unit price of the product for this line item in currency units.                                                                           |

---

## Dimension Tables
Dimension tables store descriptive attributes used to slice, filter, and group data in a fact table. Each dimension has a surrogate key (system-generated, no business meaning) that serves as the primary key and connects to main fact table, providing independence from source system identifiers.

### `gold.dim_customers`
**Description:** Stores customer identity, demographic, and geographic attributes.

**Granularity:** One row per customer.

**Sources:** `silver.crm_cust_info`, `silver.erp_cust_az12`, `silver.erp_loc_a101`

| Column          | Data Type    | Key | Description                                                                                                                              |
| --------------- | ------------ | --- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| customer_key    | INT          | PK  | Surrogate key uniquely identifying each customer in the dimension. Used as the join key in `fact_sales`.                                 |
| customer_id     | INT          |     | Natural key from the source system. Unique numerical identifier assigned to each customer. Retained for traceability back to the source. |
| customer_number | NVARCHAR(50) |     | Alphanumeric identifier used for tracking and referencing customers in the source system.                                                |
| first_name      | NVARCHAR(50) |     | Customer's first name.                                                                                                                   |
| last_name       | NVARCHAR(50) |     | Customer's last name or family name.                                                                                                     |
| country         | NVARCHAR(50) |     | Country of residence (e.g., "Australia").                                                                                                |
| marital_status  | NVARCHAR(50) |     | Marital status of the customer. Values: "Married", "Single", "N/A".                                                                      |
| gender          | NVARCHAR(50) |     | Gender of the customer. Values: "Male", "Female", "N/A".                                                                                 |
| birthdate       | DATE         |     | Date of birth formatted as YYYY-MM-DD (e.g., 1971-10-06).                                                                                |
| create_date     | DATE         |     | The date the customer record was originally created in the source system.                                                                |

### `gold.dim_products`
**Description:** Stores product attributes including categorization, pricing, and product line information. Only current product records are retained (historical versions were excluded)

**Granularity:** One row per product (current record only).

**Source:** `silver.crm_prd_info`, `silver.erp_px_cat_g1v2`

| Column         | Data Type    | Key | Description                                                                                                                                |
| -------------- | ------------ | --- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| product_key    | INT          | PK  | Surrogate key uniquely identifying each product in the dimension. Used as the join key in `fact_sales`.                                    |
| product_id     | INT          |     | Natural key from the source system. Unique identifier assigned to each product. Retained for traceability.                                 |
| product_number | NVARCHAR(50) |     | Structured alphanumeric code used for categorization or inventory tracking in the source system.                                           |
| product_name   | NVARCHAR(50) |     | Descriptive product name including details such as type, color, and size.                                                                  |
| category_id    | NVARCHAR(50) |     | Identifier for the product's high-level category classification.                                                                           |
| category       | NVARCHAR(50) |     | Broad product classification (e.g., "Bikes", "Components").                                                                                |
| subcategory    | NVARCHAR(50) |     | Detailed classification within the category (e.g., "Road Bikes", "Mountain Bikes").                                                        |
| maintenance    | NVARCHAR(50) |     | Indicates whether the product requires maintenance. Values: "Yes", "No".                                                                   |
| cost           | DECIMAL      |     | Base cost of the product in currency units.                                                                                                |
| product_line   | NVARCHAR(50) |     | Product line or series the product belongs to (e.g., "Road", "Mountain").                                                                  |
| start_date     | DATE         |     | The date the product became available for sale. Used to determine the current product record when historical versions exist in the source. |