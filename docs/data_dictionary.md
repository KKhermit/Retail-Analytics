# Raw Dataset Data Dictionary

## Scope and profiling method

Inspected on 2026-10-09. Findings come from all nine CSV files currently in `data/raw/`, using Python standard-library CSV parsing. `.gitkeep` is an empty directory placeholder, not a dataset. No source files were modified, no database tables were created, and no dependencies were added.

Counts below are exact parsed data-record counts, excluding headers, rather than estimates. CSV parsing respects quoted multiline review comments; physical line counts would overcount those records. Missing means empty or whitespace-only. Duplicate counts mean additional identical parsed rows beyond the first occurrence. Literal strings such as `null` are not automatically treated as missing.

Types are proposed relational types inferred from values and column meaning; CSV itself has no declared types. All identifier values match 32 lowercase hexadecimal characters, and all postal prefixes contain five digits. Store identifiers and postal prefixes as strings, preserving leading zeros. All nonempty timestamp values parse as `YYYY-MM-DD HH:MM:SS`; no timezone information is present, so a timezone must not be invented. Money uses exact decimal types; its currency needs confirmation from source documentation before labeling financial metrics.

## Files, row counts, and candidate keys

| File | Rows | Grain | Likely primary key | Duplicate rows |
| --- | ---: | --- | --- | ---: |
| `olist_customers_dataset.csv` | 99,441 | One order-associated customer record | `customer_id` | 0 |
| `olist_geolocation_dataset.csv` | 1,000,163 | One location observation; repeats exist | None in source; use a generated observation ID for a future raw table | 261,831 |
| `olist_order_items_dataset.csv` | 112,650 | One numbered order item | `(order_id, order_item_id)` | 0 |
| `olist_order_payments_dataset.csv` | 103,886 | One numbered payment record | `(order_id, payment_sequential)` | 0 |
| `olist_order_reviews_dataset.csv` | 99,224 | One review/order association | `(review_id, order_id)` | 0 |
| `olist_orders_dataset.csv` | 99,441 | One order | `order_id` | 0 |
| `olist_products_dataset.csv` | 32,951 | One product | `product_id` | 0 |
| `olist_sellers_dataset.csv` | 3,095 | One seller | `seller_id` | 0 |
| `product_category_name_translation.csv` | 71 | One category translation | `product_category_name` | 0 |

All proposed source keys above are nonmissing and unique in this snapshot. Geolocation has no unique postal-prefix key. Uniqueness in one snapshot is evidence for a candidate key, not a guarantee about future extracts.

## Columns

Missing counts are shown explicitly for every column. These describe this snapshot; final nullability rules should also account for source semantics.

### customers — `olist_customers_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `customer_id` | `VARCHAR(32)` | 0 | Customer record identifier used by orders; order-specific in this snapshot. |
| `customer_unique_id` | `VARCHAR(32)` | 0 | Likely persistent shopper identifier; use for repeat-customer analysis. |
| `customer_zip_code_prefix` | `VARCHAR(5)` | 0 | Five-digit postal prefix; retain leading zeros. |
| `customer_city` | `TEXT` | 0 | Customer city label. |
| `customer_state` | `VARCHAR(2)` | 0 | Two-letter customer state code. |

### geolocation — `olist_geolocation_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `geolocation_zip_code_prefix` | `VARCHAR(5)` | 0 | Five-digit postal prefix; repeated across observations. |
| `geolocation_lat` | `DOUBLE PRECISION` | 0 | Latitude in decimal degrees. |
| `geolocation_lng` | `DOUBLE PRECISION` | 0 | Longitude in decimal degrees. |
| `geolocation_city` | `TEXT` | 0 | City label; multiple labels can occur for one prefix. |
| `geolocation_state` | `VARCHAR(2)` | 0 | Two-letter state code. |

### order_items — `olist_order_items_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `order_id` | `VARCHAR(32)` | 0 | Order identifier. |
| `order_item_id` | `INTEGER` | 0 | Item sequence within an order; not globally unique. |
| `product_id` | `VARCHAR(32)` | 0 | Product identifier. |
| `seller_id` | `VARCHAR(32)` | 0 | Seller identifier. |
| `shipping_limit_date` | `TIMESTAMP WITHOUT TIME ZONE` | 0 | Recorded shipping deadline; contains a few unusually late dates. |
| `price` | `NUMERIC(12,2)` | 0 | Item price; currency is not encoded in the CSV. |
| `freight_value` | `NUMERIC(12,2)` | 0 | Item freight amount; currency is not encoded in the CSV. |

### order_payments — `olist_order_payments_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `order_id` | `VARCHAR(32)` | 0 | Order identifier. |
| `payment_sequential` | `INTEGER` | 0 | Payment sequence within an order; not globally unique. |
| `payment_type` | `TEXT` | 0 | Payment method label. |
| `payment_installments` | `INTEGER` | 0 | Recorded installment count. |
| `payment_value` | `NUMERIC(12,2)` | 0 | Payment amount; currency is not encoded in the CSV. |

### order_reviews — `olist_order_reviews_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `review_id` | `VARCHAR(32)` | 0 | Review identifier; repeated across different orders. |
| `order_id` | `VARCHAR(32)` | 0 | Order identifier. |
| `review_score` | `INTEGER` | 0 | Integer rating from 1 to 5. |
| `review_comment_title` | `TEXT` | 87,658 | Optional free-text review title; numeric-looking values remain text. |
| `review_comment_message` | `TEXT` | 58,274 | Optional free-text review message; numeric-looking values remain text. |
| `review_creation_date` | `TIMESTAMP WITHOUT TIME ZONE` | 0 | Review creation date, represented as a midnight timestamp. |
| `review_answer_timestamp` | `TIMESTAMP WITHOUT TIME ZONE` | 0 | Recorded review answer timestamp. |

### orders — `olist_orders_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `order_id` | `VARCHAR(32)` | 0 | Order identifier. |
| `customer_id` | `VARCHAR(32)` | 0 | Customer record identifier used by orders; order-specific in this snapshot. |
| `order_status` | `TEXT` | 0 | Order lifecycle status label. |
| `order_purchase_timestamp` | `TIMESTAMP WITHOUT TIME ZONE` | 0 | Recorded purchase timestamp. |
| `order_approved_at` | `TIMESTAMP WITHOUT TIME ZONE` | 160 | Recorded approval timestamp. |
| `order_delivered_carrier_date` | `TIMESTAMP WITHOUT TIME ZONE` | 1,783 | Recorded carrier handoff timestamp. |
| `order_delivered_customer_date` | `TIMESTAMP WITHOUT TIME ZONE` | 2,965 | Recorded customer delivery timestamp. |
| `order_estimated_delivery_date` | `TIMESTAMP WITHOUT TIME ZONE` | 0 | Estimated delivery date, represented as a midnight timestamp. |

### products — `olist_products_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `product_id` | `VARCHAR(32)` | 0 | Product identifier. |
| `product_category_name` | `TEXT` | 610 | Source category label; matches the translation lookup when available. |
| `product_name_lenght` | `INTEGER` | 610 | Length of product name; source spelling retained, not the name itself. |
| `product_description_lenght` | `INTEGER` | 610 | Length of product description; source spelling retained, not the description itself. |
| `product_photos_qty` | `INTEGER` | 610 | Number of product photos. |
| `product_weight_g` | `INTEGER` | 2 | Product weight in grams. |
| `product_length_cm` | `INTEGER` | 2 | Product length in centimeters. |
| `product_height_cm` | `INTEGER` | 2 | Product height in centimeters. |
| `product_width_cm` | `INTEGER` | 2 | Product width in centimeters. |

### sellers — `olist_sellers_dataset.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `seller_id` | `VARCHAR(32)` | 0 | Seller identifier. |
| `seller_zip_code_prefix` | `VARCHAR(5)` | 0 | Five-digit seller postal prefix; retain leading zeros. |
| `seller_city` | `TEXT` | 0 | Seller city label; one numeric-looking value needs investigation. |
| `seller_state` | `VARCHAR(2)` | 0 | Two-letter seller state code. |

### categories — `product_category_name_translation.csv`

| Source column | Likely SQL type | Missing rows | Meaning / notes |
| --- | --- | ---: | --- |
| `product_category_name` | `TEXT` | 0 | Source category label; matches the translation lookup when available. |
| `product_category_name_english` | `TEXT` | 0 | English translation of source category label. |

## Keys and relationships

| Child columns | Referenced columns | Observed relationship / exceptions |
| --- | --- | --- |
| `orders.customer_id` | `customers.customer_id` | All match. Both files have 99,441 distinct customer IDs; this extract is one-to-one at the customer-record grain. |
| `order_items.order_id` | `orders.order_id` | Order to many items; all match. 775 orders have no item records. |
| `order_items.product_id` | `products.product_id` | Product to many items; all match. |
| `order_items.seller_id` | `sellers.seller_id` | Seller to many items; all match. |
| `order_payments.order_id` | `orders.order_id` | Order to many payments; all match. One order has no payment records. |
| `order_reviews.order_id` | `orders.order_id` | Order to potentially many reviews; all match. 768 orders have no reviews. |
| `products.product_category_name` | Translation `product_category_name` | 610 missing categories; 13 products across two nonmissing categories have no translation. |
| `customers.customer_zip_code_prefix` | `geolocation.geolocation_zip_code_prefix` | Logical lookup only: target is not unique. 278 customer records across 157 prefixes have no match. |
| `sellers.seller_zip_code_prefix` | `geolocation.geolocation_zip_code_prefix` | Logical lookup only: target is not unique. Seven sellers across seven prefixes have no match. |

Customers have 96,096 distinct `customer_unique_id` values. There are 2,997 shopper identifiers with multiple customer records, accounting for 3,345 records beyond the first per shopper. This supports a likely shopper → customer records → orders relationship. Do not make `customer_unique_id` the primary key of the source-shaped customer table, or count `customer_id` as unique shoppers.

For items, `(order_id, order_item_id)` is unique across 112,650 rows. For payments, `(order_id, payment_sequential)` is unique across 103,886 rows. `order_id` alone is repeated for 9,803 orders in items and 2,961 orders in payments.

Reviews contain 98,410 distinct `review_id` values: 789 review IDs repeat, with 814 excess occurrences. There are 98,673 distinct reviewed orders, including 547 orders with multiple reviews. `(review_id, order_id)` is unique across all 99,224 rows and preserves the observed grain. A generated review-row key is an alternative for future extracts if this composite key stops being unique; it adds another identifier without solving source duplication by itself.

## Data-quality findings

### Missing values and incomplete records

- Orders: 160 missing approval timestamps, 1,783 missing carrier handoff timestamps, and 2,965 missing customer delivery timestamps. Missing lifecycle events may be expected for some statuses; they should be evaluated by status rather than dropping every incomplete order.
- Even among `delivered` orders, 14 lack approval timestamps, two lack carrier handoff timestamps, and eight lack customer delivery timestamps. These groups can overlap.
- Products: 610 rows lack category, name length, description length, and photo count; two rows lack weight and each dimension.
- Reviews: 87,658 missing titles (88.34%) and 58,274 missing messages (58.73%). Optional comments should remain nullable; a rating without text is still usable.
- Customers, sellers, geolocation, items, payments, and category translations have no empty or whitespace-only fields. This does not imply all their values are valid.

### Repeated rows, location ambiguity, and join risk

- Geolocation contains 1,000,163 rows and 261,831 exact duplicate rows, leaving 738,332 distinct rows. Its 19,015 postal prefixes are not unique: 17,972 appear more than once.
- 8,556 postal prefixes have multiple city labels and eight have multiple state labels. Some city variation may be spelling or accent variation; some needs investigation. One geolocation city value has surrounding whitespace.
- No other CSV has exact duplicate rows. Repeated customer, order, or review identifiers described above must be interpreted at their correct grain.
- Joining geolocation directly to customers or sellers on postal prefix multiplies rows. Likewise, joining items, payments, and reviews together by order can multiply money amounts. Aggregate each child table to the required grain before combining order-level metrics.

### Unmatched categories and locations

- The 71-row translation file does not cover `pc_gamer` and `portateis_cozinha_e_preparadores_de_alimentos` (13 products combined). Products have 73 distinct nonmissing category labels.
- Preserve unmatched categories with a nullable English translation in a future category lookup; do not discard their products or invent translations.
- Postal coverage is incomplete for customers and sellers. Use optional location enrichment rather than dropping unmatched records or enforcing a foreign key directly to the raw geolocation file.

### Suspicious values and temporal anomalies

- Purchases range from `2016-09-04 21:15:19` to `2018-10-17 17:30:18`. Four item shipping deadlines are after 2018, with a maximum of `2020-04-09 22:35:08`; investigate before using them in fulfillment metrics.
- 1,359 orders have carrier handoff before approval, and 23 have customer delivery before carrier handoff (comparisons require both timestamps). No observed approval timestamp precedes purchase. Treat these as investigation flags rather than silently correcting dates.
- Geolocation latitude ranges from -36.6053744107061 to 45.06593318269697; longitude ranges from -101.46676644931476 to 121.10539381057764. All are within global coordinate limits, but some are implausible for this apparent Brazilian dataset. A rough screening box (latitude -34 to 6, longitude -74 to -34) flags 42 rows; this is a heuristic, not a country-boundary validation.
- One seller city is `04482255`, a numeric-looking value that is suspicious for a city label. Keep it as text and investigate.
- Payment methods include `credit_card`, `boleto`, `voucher`, `debit_card`, and `not_defined` (three records). Nine payment records have zero value and two have zero installments. These need interpretation; zero is not automatically missing.
- Four products have zero weight; 383 items have zero freight. Zero freight may be legitimate. Item prices are positive, and all observed money values are nonnegative.
- Ratings are all integers from 1 to 5. Item sequences range from 1 to 21; payment sequences from 1 to 29; installments from 0 to 24.
- Review titles have surrounding whitespace in 1,998 rows and messages in 9,451 rows. One message is a literal missing-token string (`null`, `none`, `nan`, or `n/a`, case-insensitive); retain raw text until a cleaning policy is chosen.
- Source columns `product_name_lenght` and `product_description_lenght` contain spelling errors. Preserve their names in raw ingestion; explicitly map to `product_name_length` and `product_description_length` in a cleaned layer later.

These are profiling findings, not a complete business-rule audit. Currency, timezone, provenance, detailed status semantics, payment reconciliation, and geographical accuracy still need confirmation. No cleaning or deduplication has been applied.

## Recommended logical relational schema

Start with source-shaped relational tables rather than a dimensional warehouse. This keeps ingestion understandable and preserves the dataset grain. The following are recommendations only; no tables or SQL definitions have been created.

| Recommended table | Primary key | Foreign keys / design notes |
| --- | --- | --- |
| `customers` | `customer_id` | Keep `customer_unique_id` as a nonunique shopper identifier, plus source location fields. |
| `orders` | `order_id` | `customer_id` → `customers`; retain status and nullable lifecycle timestamps. |
| `products` | `product_id` | Nullable category label → proposed `product_categories` once all observed categories are included. Keep product attributes. |
| `sellers` | `seller_id` | Keep source postal prefix, city, and state. |
| `order_items` | `(order_id, order_item_id)` | References `orders`, `products`, and `sellers`; keep shipping deadline, price, and freight. |
| `order_payments` | `(order_id, payment_sequential)` | References `orders`; keep method, installments, and amount. |
| `order_reviews` | `(review_id, order_id)` | References `orders`; keep score, optional comments, and timestamps. |
| `product_categories` | `product_category_name` | Union of translation keys and nonmissing product categories (73 labels here); English translation nullable for two categories. |
| `geolocation_observations` | Generated `geolocation_observation_id` | Preserve source location rows, including duplicates; postal prefix is a nonunique lookup field. |

Using only the 71-row translation table as a strict category parent is simpler to ingest but would reject 13 products. Building a 73-label category lookup preserves all products while making missing translations explicit. Any such enrichment must occur in a derived table, leaving raw files unchanged.

A future derived `postal_prefix_locations` table could have one row per postal prefix for safe enrichment. It would require an explicit policy for duplicate observations, conflicting city/state labels, coordinate outliers, and representative coordinates. Do not create it by arbitrarily selecting the first raw row. Keep raw customer/seller postal fields without a strict location foreign key until coverage and conflict rules are resolved.

A separate `shoppers` table keyed by `customer_unique_id` is an optional later normalization. For the first relational load, retaining both customer identifiers in `customers` is simpler and preserves order-associated location snapshots without assigning one address to a repeat shopper.

For the next implementation step, create the core parent tables (`customers`, `products`, `sellers`) and `orders`, then their child tables (`order_items`, `order_payments`, `order_reviews`). Add category lookup handling before enforcing its foreign key; handle geolocation separately because it needs different key and enrichment rules. Final constraints and load order should be agreed when database implementation is requested.
