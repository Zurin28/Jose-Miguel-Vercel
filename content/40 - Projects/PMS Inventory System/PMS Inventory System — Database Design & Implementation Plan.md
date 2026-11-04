---
title: PMS Inventory System — Database Design & Implementation Plan
tags:
  - project
  - database
  - ads
  - schema
  - mysql
  - normalization
  - api-specification
created: 2026-11-03
status: active
client: PMS School and Office Supplies Trading
system_type: Single-user Web Inventory & Delivery Monitoring
architecture: Zero-Framework PHP + MySQL 8.0 InnoDB + Vanilla JS / CSS
codebase_path: "C:/Users/ASUS/Documents (ASUS)/PROGRAMMING/WMSU PROJECTS/ADS PMS TRADING INVENTORY SYSTEM"
---

# 🗄️ PMS Inventory System — Database Design & Implementation Plan

Comprehensive technical knowledge base entry synthesizing the formal **Advanced Database Systems (ADS)** specification, Enhanced Entity-Relationship (EER) modeling, relational schema, UNF-to-4NF normalization proofs, SQL DDL/DML, and server-side zero-framework API architecture for the **PMS School and Office Supplies Trading** inventory tracking and delivery monitoring system.

---

## 📌 Executive Summary & Architectural Philosophy

* **Target Business:** PMS School and Office Supplies Trading — single-owner, single-branch trading business selling school, office, and janitorial supplies.
* **Target User:** Owner only (Single-user operational model; customers are records, not system users).
* **Core Philosophy:** **Zero-Framework Architecture** — zero Composer packages, no heavy frameworks (no Laravel, Tailwind, React, Bootstrap, or jQuery). Written in clean **PHP 8 (PDO prepared statements)**, **MySQL 8.0 InnoDB**, and **Vanilla JS / CSS**.
* **Document Generation:** Delivery receipts generated via native browser print CSS (`window.print()` to PDF) and built-in PHP `ZipArchive` for lightweight `.docx` and `.pptx` generation.
* **Core Domain Problems Addressed:**
  1. Elimination of manual physical stock cards.
  2. Automated real-time Stock On Hand ($SOH$) derivation without drift.
  3. Unified tracking of delivery receipts and office balance receivables.
  4. Real-time KPI dashboard (revenue, collected, outstanding debt, low stock).

---

## 📐 EER Modeling & Relational Schema (15 Tables)

### Entity Types & Subtype Specialization
* **Supertype/Subtype Hierarchy:**
  * `customer` (Supertype: `customer_id`, `name`, `address`, `customer_type` [`SCHOOL` | `OFFICE`])
    * `school` (Subtype: `customer_id` PK/FK, `school_level`, `head_name`)
    * `office` (Subtype: `customer_id` PK/FK, `office_type`, `contact_person`)
  * `customer_contact` (Multivalued contact points: `customer_id`, `contact_type` [`MOBILE`|`LANDLINE`|`EMAIL`], `contact_value`)
  * `delivery` (Supertype: `delivery_id`, `receipt_no`, `customer_id`, `delivery_date`, `order_type` [`DIRECT` | `BIDDING`], `status`)
    * `direct_delivery` (Subtype: `delivery_id` PK/FK, `po_number`, `po_date`)
    * `bidding_delivery` (Subtype: `delivery_id` PK/FK, `bid_ref_no`, `delivery_deadline`)

### Relational Table Manifest & Referential Rules

| Table | Primary Key | Foreign Keys | Key Constraints & Integrity |
| :--- | :--- | :--- | :--- |
| `category` | `category_id` | *None* | `name` UNIQUE |
| `item` | `item_id` | `category_id` $\to$ `category` | `item_code` UNIQUE; (`name`, `unit`) UNIQUE; `unit_price` $\ge 0$ |
| `item_price_history`| (`item_id`, `effective_at`) | `item_id` $\to$ `item` (CASCADE) | Audit trail of unit price changes |
| `stock_in` | `stock_in_id` | `item_id` $\to$ `item` (RESTRICT) | `quantity` $> 0$; Index on (`item_id`, `date_received`) |
| `stock_adjustment` | `adjustment_id` | `item_id` $\to$ `item` (RESTRICT) | Discrepancy reconciliation between physical count and system |
| `customer` | `customer_id` | *None* | (`name`, `address`) UNIQUE |
| `school` | `customer_id` | `customer_id` $\to$ `customer` (CASCADE) | `school_level` ENUM |
| `office` | `customer_id` | `customer_id` $\to$ `customer` (CASCADE) | `office_type` ENUM |
| `customer_contact` | (`customer_id`, `contact_type`, `contact_value`) | `customer_id` $\to$ `customer` (CASCADE) | 4NF-compliant contact storage |
| `delivery` | `delivery_id` | `customer_id` $\to$ `customer` (RESTRICT)| `receipt_no` UNIQUE; Index on `delivery_date` |
| `direct_delivery` | `delivery_id` | `delivery_id` $\to$ `delivery` (CASCADE) | `po_number`, `po_date` |
| `bidding_delivery` | `delivery_id` | `delivery_id` $\to$ `delivery` (CASCADE) | `bid_ref_no`, `delivery_deadline` |
| `delivery_item` | (`delivery_id`, `item_id`) | `delivery_id` (CASCADE), `item_id` (RESTRICT) | `unit_price` immutable snapshot at delivery time |
| `payment` | `payment_id` | `delivery_id` $\to$ `delivery` (RESTRICT)| Tied strictly to specific delivery; `amount` $> 0$ |
| `user_account` | `user_id` | *None* | Single-row singleton (`CHECK (user_id = 1)`), password hash |

---

## 🧪 Formal Database Normalization (UNF to BCNF / 4NF)

1. **Unnormalized Form (UNF):** Flat paper delivery receipts containing customer details, repeated delivery line items, multiple contact numbers, and multiple payments on one sheet.
2. **First Normal Form (1NF):** Eliminate repeating groups. Atomic values for delivery header, items, payments, and customer contacts.
3. **Second Normal Form (2NF):** Eliminate partial dependencies on composite keys. Extracted `item` facts from composite line items (`receipt_no`, `item_code`).
4. **Third Normal Form (3NF):** Eliminate transitive dependencies.
   * `receipt_no` $\to$ `customer_name` $\to$ `customer_address`: Extracted `customer` entity.
   * `category_name` extracted to `category` table to prevent anomaly upon category renaming.
   * Introduced subtype decomposition (`direct_delivery` vs `bidding_delivery`, `school` vs `office`) to eliminate NULL-heavy records.
5. **Boyce-Codd Normal Form (BCNF):** Ensured all determinants are candidate keys. Replaced natural keys with surrogate auto-increment IDs (`customer_id`, `item_id`, `delivery_id`) to prevent cascading rename anomalies while maintaining UNIQUE candidate keys.
6. **Fourth Normal Form (4NF):** Eliminate multivalued dependencies ($MVD$). Multiple contact numbers/emails decoupled from customer entity into `customer_contact(customer_id, contact_type, contact_value)`.

---

## ⚙️ Views, Triggers, and Analytical Engine

### Core SQL Views
```sql
-- Unified Ledger of Stock Movements
CREATE VIEW v_stock_movement AS
SELECT item_id, date_received AS move_date, 'IN' AS move_type, CAST(quantity AS SIGNED) AS qty 
FROM stock_in
UNION ALL
SELECT di.item_id, d.delivery_date, 'OUT', -CAST(di.quantity AS SIGNED)
FROM delivery_item di 
JOIN delivery d ON d.delivery_id = di.delivery_id 
WHERE d.status = 'DELIVERED'
UNION ALL
SELECT item_id, adjust_date, 'ADJ', CAST(counted_qty AS SIGNED) - system_qty 
FROM stock_adjustment;

-- Real-time Stock On Hand (SOH) Computation
CREATE VIEW v_item_stock AS
SELECT i.item_id, i.item_code, i.name, c.name AS category, i.unit, i.unit_price, i.low_stock_level, i.status,
       COALESCE(m.soh, 0) AS stock_on_hand
FROM item i 
JOIN category c ON c.category_id = i.category_id
LEFT JOIN (SELECT item_id, SUM(qty) AS soh FROM v_stock_movement GROUP BY item_id) m ON m.item_id = i.item_id;

-- Delivery Balance & Debt Monitoring
CREATE VIEW v_delivery_balance AS
SELECT d.delivery_id, d.receipt_no, d.customer_id, d.delivery_date, d.status,
       COALESCE(t.total, 0) AS total_amount, 
       COALESCE(p.paid, 0) AS total_paid,
       COALESCE(t.total, 0) - COALESCE(p.paid, 0) AS balance
FROM delivery d
LEFT JOIN (SELECT delivery_id, SUM(quantity * unit_price) AS total FROM delivery_item GROUP BY delivery_id) t ON t.delivery_id = d.delivery_id
LEFT JOIN (SELECT delivery_id, SUM(amount) AS paid FROM payment GROUP BY delivery_id) p ON p.delivery_id = d.delivery_id;
```

### Business Rule Enforcement Triggers
* **`trg_di_stock_check` (BEFORE INSERT ON delivery_item):** Blocks delivery line creation if requested quantity exceeds current `stock_on_hand` (`SIGNAL SQLSTATE '45000'`).
* **`trg_si_delete_guard` (BEFORE DELETE ON stock_in):** Prevents stock-in deletion if retroactive removal would drop `stock_on_hand` below zero.
* **`trg_bd_office_only` (BEFORE INSERT ON bidding_delivery):** Guarantees bidding orders are strictly linked to customers with `customer_type = 'OFFICE'`.
* **`trg_payment_no_overpay` (BEFORE INSERT ON payment):** Rejects payments exceeding the outstanding delivery balance.
* **`trg_item_price_ins` & `trg_item_price_log`:** Automatically tracks price changes into `item_price_history`.

---

## 🌐 Zero-Framework API & Frontend Contract

### System Directory Layout
```text
pms/
├── index.html                   # Entry redirect
├── login.html                   # Auth UI
├── dashboard.html               # Live KPI counters
├── items.html                   # Inventory catalog & low-stock tags
├── stock_in.html                # Restock logger
├── stock_card.html              # Stock card audit ledger
├── customers.html               # Schools and Offices directory
├── deliveries.html              # Delivery log & office debt filters
├── delivery_new.html            # Order fulfillment form (Direct/Bidding)
├── receipt.html                 # Printable delivery receipt (CSS print)
├── payments.html                # Collection & balance payment entry
├── count.html                   # Physical inventory audit adjustment
├── reports.html                 # Period-based movement & revenue analytics
├── css/
│   └── app.css                  # Pure CSS variables, grid, flexbox, print layout
├── js/
│   ├── api.js                   # Unified fetch client (401 interceptor)
│   ├── ui.js                    # DOM tables, toasts, form handling
│   └── [page].js                # Modular per-page scripts (items.js, deliveries.js, etc.)
├── api/
│   ├── _bootstrap.php           # Session check, JSON responders, error mapper
│   ├── auth.php                 # login, logout, me
│   ├── items.php                # Item CRUD & archiving
│   ├── categories.php           # Category management
│   ├── customers.php            # Customer hierarchy & contact CRUD
│   ├── stock_in.php             # Inbound supply logging
│   ├── stock_card.php           # Historical movement ledger
│   ├── deliveries.php           # Atomic multi-table delivery transactions
│   ├── payments.php             # Payment receipts & overpay guards
│   ├── adjustments.php          # Physical count variance recorder
│   ├── dashboard.php            # Aggregated metrics
│   ├── reports.php              # Periodic beginning/in/out/ending matrix
│   └── receipt.php              # HTML, DOCX, and PPTX generator
└── config/
    └── db.php                   # Singleton PDO MySQL connector (Ignored in VCS)
```

---

## 🚦 Verified Test & Rollout Sequence

1. **Step 1: Database & Seed Execution:** Run `01_schema.sql`, `02_views.sql`, `03_triggers.sql`, `04_seed.sql`. Verify 15 tables and views via `SHOW TABLES;`.
2. **Step 2: Core Auth & Connection:** Verify `config/db.php`, `api/_bootstrap.php`, and `api/auth.php` via cURL.
3. **Step 3: Resource CRUD:** Execute automated cURL test suite against `api/items.php` (test 401 unauth, 200 list, 201 insert, 409 duplicate code, 200 price update trigger, 200 delete/archive).
4. **Step 4: Transactional Delivery:** Validate atomic insertion across `delivery`, `direct_delivery` / `bidding_delivery`, and `delivery_item`, asserting that insufficient stock aborts with HTTP 409.
5. **Step 5: Document & Print:** Verify receipt layout in browser print dialog; optionally test ZipArchive `.docx`/`.pptx` output.

---

## 🔗 Related Documentation & Links
* 📦 **[[PMS Inventory System - Project Architecture & Requirements Proposal]]** — Core business domain, client problem context, and delivery requirements.
* 🚀 **[[Projects & Software Development (Map of Content)]]** — Master software projects directory.
* 💻 **Active Codebase Location:** [`ADS PMS TRADING INVENTORY SYSTEM`](file:///C:/Users/ASUS/Documents%20(ASUS)/PROGRAMMING/WMSU%20PROJECTS/ADS%20PMS%20TRADING%20INVENTORY%20SYSTEM)
* 📂 **Source Code Reference:** `C:\Users\ASUS\Downloads\pms_inventory_system_code_proper_sample.md`
* 📄 **ADS Specification Reference:** `C:\Users\ASUS\Downloads\PMS_ADS_Database_Design_and_Implementation_Plan.pdf`
