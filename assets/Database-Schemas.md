# 🗄️ Vedashi Database Schemas

Welcome to the Vedashi Database Schema documentation. This document outlines the *actual* core data models and their relationships as fetched directly from our PostgreSQL database (`inventory` schema). 

These schemas form the backbone of our enterprise e-commerce operations, specifically focusing on the **Supplier**, **Product**, **Order**, and **Order Item** entities.

---

## 1. Supplier Schema (`suppliers`)

*Note: In the Vedashi architecture, vendors/merchants are primarily represented as `suppliers` who provide goods for the platform.*

| Field Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID | Primary Key | Unique identifier for the supplier. |
| `name` | String (Varchar) | Required | The public-facing name of the supplier. |
| `contact_person` | String (Varchar) | Optional | Name of the primary contact person. |
| `email` | String (Varchar) | Optional | Primary email for business inquiries. |
| `phone` | String (Varchar) | Optional | Primary contact phone number. |
| `address` | Text | Optional | Full physical address. |
| `country` | String (Varchar) | Optional | Country of origin/operation. |
| `lead_time_days` | Integer | Optional | Estimated time required for fulfillment. |
| `rating` | Numeric | Default: 0 | Supplier performance rating. |
| `is_active` | Boolean | Default: true | Status indicating if the supplier is currently active. |
| `created_at` | Timestamp | Auto | Record creation timestamp. |

---

## 2. Product Schema (`products`)

The Product model represents individual base items listed for sale. *(Note: Specific pricing and inventory are managed in a separate `product_variants` table).*

| Field Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `product_id` | UUID | Primary Key | Unique identifier for the product. |
| `sku` | Text | Required | Stock Keeping Unit identifier. |
| `product_name` | Text | Required | The display name of the product. |
| `brand` | Text | Optional | Brand associated with the product. |
| `description` | Text | Optional | Detailed HTML/Text description of the product. |
| `category_id` | UUID | Foreign Key | References the `categories` table. |
| `status` | Text | Default: 'active' | Product status (e.g., active, draft). |
| `product_type` | String (Varchar) | Optional | General classification of the product. |
| `avg_rating` | Numeric | Default: 0 | Average customer rating. |
| `slug` | String (Varchar) | Optional | URL-friendly identifier for the product page. |
| `is_featured` | Boolean | Default: false| Indicates if the product is featured on the storefront. |
| `search_vector` | TSVector | Optional | Full-text search optimized vector. |
| `created_at` | Timestamp | Auto | Record creation timestamp. |

---

## 3. Order Schema (`orders`)

The Order model records all customer purchases and tracks fulfillment states.

| Field Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `order_id` | UUID | Primary Key | Unique identifier for the order. |
| `customer_id` | UUID | Foreign Key | References the `customers` table. |
| `total_amount` | Numeric | Required | Total initial order value. |
| `order_status` | Text | Default: 'PENDING' | Fulfillment state (e.g., PENDING, SHIPPED). |
| `payment_status` | Text | Default: 'UNPAID' | Financial state (e.g., UNPAID, PAID). |
| `payment_method` | Text | Default: 'cod' | e.g., 'cod', 'razorpay'. |
| `discount_amount`| Numeric | Default: 0 | Total discount applied (sale + coupon). |
| `shipping_amount`| Bigint | Default: 0 | Calculated shipping cost. |
| `final_total` | Bigint | Default: 0 | Final payable amount. |
| `currency` | Text | Default: 'VND'| The currency of the transaction. |
| `shipping_address_id`| UUID | Foreign Key | References `customer_addresses` table. |
| `billing_address_id` | UUID | Foreign Key | References `customer_addresses` table. |
| `created_at` | Timestamp | Auto | Record creation timestamp. |

---

## 4. Order Items Mapping (`order_items`)

This mapping table records the exact state of a product variant at the time of purchase to ensure historical accuracy even if the base product changes.

| Field Name | Data Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `order_item_id` | UUID | Primary Key | Unique identifier for the order line item. |
| `order_id` | UUID | Foreign Key | References the `orders` table. |
| `variant_id` | UUID | Foreign Key | References the `product_variants` table. |
| `quantity` | Integer | Required | Number of units purchased. |
| `unit_price` | Numeric | Required | Price per unit *at the time of purchase*. |
| `product_name_snapshot`| Text | Optional | Immutable record of the product name at purchase. |
| `variant_name_snapshot`| Text | Optional | Immutable record of the variant details at purchase. |
| `sku_snapshot` | Text | Optional | Immutable record of the SKU at purchase. |

---

## 🔗 Entity Relationships Highlights

- **Supplier to Products (1:N):** Handled via `product_suppliers` junction tables or direct `supplier_id` on variants, connecting products to their sources.
- **Product to Variants (1:N):** A single Product can have many `product_variants` (which hold actual price and inventory).
- **Order to Variants (M:N):** An Order contains many variants. This is resolved via the `order_items` mapping table, which takes snapshots of text fields (like name and SKU) to ensure historical integrity.
- **Customer to Orders (1:N):** A `customer` places multiple `orders`. Address details are strictly normalized via `shipping_address_id` and `billing_address_id` pointing to the `customer_addresses` table.
