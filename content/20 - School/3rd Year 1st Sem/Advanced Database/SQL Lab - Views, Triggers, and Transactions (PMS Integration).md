---
title: "SQL Lab: Views, Triggers, and Stored Procedures with Transactions"
tags:
  - academic
  - advanced-database
  - wmsu
  - sql
  - mariadb
  - triggers
  - views
  - stored-procedures
  - transactions
  - pms-inventory
date: 2026-11-05
course: Advanced Database Systems (ADS)
institution: Western Mindanao State University (WMSU)
target_system: "[[PMS Inventory System - Database Design & Implementation Plan|PMS Trading Inventory System]]"
---

# SQL Lab: Views, Triggers, and Stored Procedures with Transactions

## Executive Overview

This technical lab summary documents the MariaDB/MySQL coding session conducted in **Advanced Database Systems (ADS)** under XAMPP. The session covered three database architectural concepts requested by the instructor for integration into the **PMS Trading Inventory System**:
1. **Relational Views:** Abstracting complex entity relationships and joins for clean read interfaces.
2. **Automated Audit Triggers:** Capturing state modifications (`OLD` vs. `NEW`) into append-only audit tables.
3. **Stored Procedures with ACID Transactions:** Enforcing atomicity, consistency, and pessimistic row locking (`FOR UPDATE`) with explicit rollback mechanisms.

---

## 1. Environment & Connection Setup

### 1.1 CLI Login Nuance
When launching the MariaDB CLI from the Windows Command Prompt/PowerShell inside `c:\xampp`:
```bash
# INCORRECT: Trailing semicolon included in CLI command
mysql -u root;
# Output: ERROR 1045 (28000): Access denied for user 'root;'@'localhost'

# CORRECT: Standard invocation
mysql -u root
```
> [!NOTE]
> Including `;` inside the OS shell argument makes MariaDB interpret the username as `root;` literally, triggering an authentication failure.

### 1.2 Target Database
```sql
USE ads_views_and_triggers;
```

---

## 2. Part 1: Relational Views (Data Abstraction)

### 2.1 Table Schemas & Seed Data
The lab used an HR model (`departments` and `employees`) demonstrating `FOREIGN KEY` constraints and nullable relationships:

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary DECIMAL(10,2),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

INSERT INTO departments (department_id, department_name) VALUES
(1, 'Engineering'),
(2, 'Human Resources'),
(3, 'Finance'),
(4, 'Marketing');

INSERT INTO employees (employee_id, first_name, last_name, salary, department_id) VALUES
(101, 'Alex', 'Rivera', 85000.00, 1),
(102, 'Jordan', 'Lee', 92000.00, 1),
(103, 'Morgan', 'Smith', 62000.00, 2),
(104, 'Taylor', 'Chen', 74000.00, 3),
(105, 'Casey', 'Patel', 58000.00, NULL);
```

### 2.2 View Definition: `v_employee_directory`
Views encapsulate table joins and format fields at the database level, preventing repetitive application-level SQL:

```sql
CREATE VIEW v_employee_directory AS
SELECT
    e.employee_id,
    CONCAT(e.first_name, ' ', e.last_name) AS full_name,
    d.department_name
FROM
    employees e
LEFT JOIN
    departments d ON e.department_id = d.department_id;
```

### 2.3 Query & Verification
```sql
SELECT *
FROM v_employee_directory
WHERE department_name = 'Engineering'
ORDER BY full_name ASC;
```

**Result:**
| employee_id | full_name | department_name |
| :--- | :--- | :--- |
| `101` | Alex Rivera | Engineering |
| `102` | Jordan Lee | Engineering |

---

## 3. Part 2: Triggers & Inventory Audit Trail

### 3.1 Tables
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    stock_quantity INT
);

CREATE TABLE inventory_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT,
    old_quantity INT,
    new_quantity INT,
    action_type VARCHAR(20),
    change_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO products (product_id, product_name, stock_quantity) VALUES
(1, 'Laptop', 25),
(2, 'Wireless Mouse', 100);
```

### 3.2 The CLI Delimiter Trap & Resolution
When writing compound procedural statements (`BEGIN ... END`) in the MySQL/MariaDB CLI, the default statement terminator `;` causes premature execution parsing errors:

> [!WARNING]
> Running multi-line trigger blocks without `DELIMITER //` yields `ERROR 1064 (42000): You have an error in your SQL syntax near '' at line 19` because the client sends the statement before reaching the closing `END`.

**Proper Delimiter-Wrapped Trigger Definition:**
```sql
DELIMITER //

CREATE TRIGGER trg_after_inventory_update
AFTER UPDATE ON products
FOR EACH ROW
BEGIN
    IF OLD.stock_quantity <> NEW.stock_quantity THEN
        INSERT INTO inventory_audit (
            product_id,
            old_quantity,
            new_quantity,
            action_type,
            change_time
        )
        VALUES (
            OLD.product_id,
            OLD.stock_quantity,
            NEW.stock_quantity,
            'STOCK_UPDATE',
            NOW()
        );
    END IF;
END //

DELIMITER ;
```

### 3.3 Verification & Output Validation
```sql
-- Transaction 1: Decrement stock by 5
UPDATE products
SET stock_quantity = stock_quantity - 5
WHERE product_id = 1;

-- Transaction 2: Increment stock by 10
UPDATE products
SET stock_quantity = stock_quantity + 10
WHERE product_id = 1;

-- Inspect audit log
SELECT * FROM inventory_audit;
```

**Verification Results:**
| audit_id | product_id | old_quantity | new_quantity | action_type | change_time |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `1` | `1` | `25` | `20` | `STOCK_UPDATE` | 2026-11-05 11:36:58 |
| `2` | `1` | `20` | `30` | `STOCK_UPDATE` | 2026-11-05 11:47:10 |

---

## 4. Part 3: Stored Procedures & ACID Transactions

### 4.1 Schema Setup
```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_holder VARCHAR(100),
    balance DECIMAL(10, 2)
);

INSERT INTO accounts (account_id, account_holder, balance) VALUES
(1, 'Alice Johnson', 1000.00),
(2, 'Bob Smith', 250.00);
```

### 4.2 Stored Procedure: `sp_transfer_funds`
Demonstrates concurrency control, pessimistic locking, atomic transactions, and custom error signaling:

```sql
DELIMITER //

CREATE PROCEDURE sp_transfer_funds(
    IN p_sender_id INT,
    IN p_receiver_id INT,
    IN p_amount DECIMAL(10, 2)
)
BEGIN
    DECLARE v_sender_balance DECIMAL(10, 2);

    -- 1. Begin atomic boundary
    START TRANSACTION;

    -- 2. Concurrency Control: Exclusive row-level lock
    SELECT balance INTO v_sender_balance
    FROM accounts
    WHERE account_id = p_sender_id
    FOR UPDATE;

    -- 3. Business rule validation
    IF v_sender_balance >= p_amount THEN
        -- Deduct from sender
        UPDATE accounts
        SET balance = balance - p_amount
        WHERE account_id = p_sender_id;

        -- Credit receiver
        UPDATE accounts
        SET balance = balance + p_amount
        WHERE account_id = p_receiver_id;

        -- Persist changes
        COMMIT;
        SELECT 'Transfer completed successfully' AS status_message;
    ELSE
        -- Abort and roll back any dirty reads/writes
        ROLLBACK;
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Transaction failed: Insufficient balance';
    END IF;
END //

DELIMITER ;
```

### 4.3 Inspection & Execution
```sql
-- Valid inspection command in MariaDB:
SHOW PROCEDURE STATUS WHERE Db = 'ads_views_and_triggers';
-- Note: 'SHOW PROCEDURES;' is invalid syntax in MySQL/MariaDB

-- Execute transfer: Move 200.00 from Alice (1) to Bob (2)
CALL sp_transfer_funds(1, 2, 200.00);

-- Confirm account balances
SELECT * FROM accounts;
```

**Post-Transfer State:**
| account_id | account_holder | balance |
| :--- | :--- | :--- |
| `1` | Alice Johnson | `800.00` |
| `2` | Bob Smith | `450.00` |

---

## 5. Direct Integration Blueprint: PMS Trading Inventory System

The professor required mapping these three mechanisms directly into the codebase under `ADS PMS TRADING INVENTORY SYSTEM`:

| Lab Demonstration Concept | Equivalent PMS Trading Requirement | PMS Target SQL File |
| :--- | :--- | :--- |
| `CREATE VIEW v_employee_directory` | Real-time Stock on Hand (`v_item_stock`), Delivery Balances (`v_delivery_balance`) | `sql/02_views.sql` |
| `trg_after_inventory_update` (Audit log) | Automated `item_audit` or `stock_ledger` recording all stock shifts & price alterations | `sql/03_triggers.sql` |
| `sp_transfer_funds` (ACID + `FOR UPDATE`) | Atomic Payment Processing (`sp_record_delivery_payment`) or Stock Disbursement | `sql/06_procedures.sql` *(New)* |

### 5.1 PMS Production Component 1: Automated Inventory Audit Trigger
To track all item stock modifications automatically in PMS Trading:

```sql
-- Schema addition for PMS
CREATE TABLE IF NOT EXISTS item_stock_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    item_id INT NOT NULL,
    old_stock INT NOT NULL,
    new_stock INT NOT NULL,
    changed_by INT NULL,
    change_type ENUM('STOCK_IN', 'DELIVERY', 'ADJUSTMENT') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (item_id) REFERENCES item(item_id) ON DELETE CASCADE
);

DELIMITER //

CREATE TRIGGER trg_item_stock_audit
AFTER UPDATE ON item
FOR EACH ROW
BEGIN
    IF OLD.stock_on_hand <> NEW.stock_on_hand THEN
        INSERT INTO item_stock_audit (
            item_id,
            old_stock,
            new_stock,
            change_type,
            created_at
        ) VALUES (
            OLD.item_id,
            OLD.stock_on_hand,
            NEW.stock_on_hand,
            IF(NEW.stock_on_hand > OLD.stock_on_hand, 'STOCK_IN', 'DELIVERY'),
            NOW()
        );
    END IF;
END //

DELIMITER ;
```

### 5.2 PMS Production Component 2: Stored Procedure for Safe Payment Processing
Replaces raw multi-query PHP execution with an atomic stored procedure enforcing row-level locks on deliveries:

```sql
DELIMITER //

CREATE PROCEDURE sp_record_delivery_payment(
    IN p_delivery_id INT,
    IN p_amount DECIMAL(12, 2),
    IN p_payment_method VARCHAR(50),
    IN p_reference_no VARCHAR(100),
    IN p_received_by INT
)
BEGIN
    DECLARE v_balance DECIMAL(12, 2);
    DECLARE v_status VARCHAR(20);

    START TRANSACTION;

    -- Pessimistic lock on the delivery record
    SELECT status, balance INTO v_status, v_balance
    FROM v_delivery_balance
    WHERE delivery_id = p_delivery_id
    FOR UPDATE;

    -- Validate payment rules
    IF v_balance IS NULL THEN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Delivery not found';
    ELSEIF p_amount <= 0 THEN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Payment amount must be greater than zero';
    ELSEIF p_amount > v_balance THEN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Payment exceeds outstanding delivery balance';
    ELSE
        -- 1. Insert payment record
        INSERT INTO payment (
            delivery_id,
            amount,
            payment_date,
            payment_method,
            reference_no,
            received_by
        ) VALUES (
            p_delivery_id,
            p_amount,
            CURDATE(),
            p_payment_method,
            p_reference_no,
            p_received_by
        );

        -- 2. If balance reaches zero, update delivery payment status
        IF (v_balance - p_amount) = 0 THEN
            UPDATE delivery
            SET status = 'PAID'
            WHERE delivery_id = p_delivery_id;
        END IF;

        COMMIT;
        SELECT 'Payment recorded successfully' AS message, (v_balance - p_amount) AS remaining_balance;
    END IF;
END //

DELIMITER ;
```

---

## 6. Diagnostic Troubleshooting & Syntax Cheatsheet

| Error Code | Error Message | Root Cause | Resolution |
| :--- | :--- | :--- | :--- |
| `ERROR 1045` | Access denied for user `'root;'@'localhost'` | CLI flag included `;` at the end | Run `mysql -u root` without semicolon |
| `ERROR 2013 / 2006` | Lost connection / MySQL server gone away | CLI timeout or server restart during query | Client auto-reconnects on next valid command |
| `ERROR 1064` | Syntax error near `'' at line 19` / `'END IF'` | Default delimiter `;` conflicts with procedural block | Enclose statement in `DELIMITER // ... END // DELIMITER ;` |
| `ERROR 1064` | Syntax error near `'procedures'` | `SHOW PROCEDURES;` is not valid MariaDB syntax | Use `SHOW PROCEDURE STATUS;` |
| `ERROR 1359` | Trigger already exists | Re-creating an existing trigger without dropping | Execute `DROP TRIGGER IF EXISTS trg_name;` before `CREATE TRIGGER` |
| `ERROR 1062` | Duplicate entry `'1'` for key `'PRIMARY'` | Re-inserting existing primary key ID | Verify current table contents before re-running seed scripts |

---

## 7. Next Actions for PMS Project

1. Add `sql/06_procedures.sql` to `ADS PMS TRADING INVENTORY SYSTEM` containing `sp_record_delivery_payment` and `sp_process_delivery`.
2. Ensure `DELIMITER $$` or `DELIMITER //` is documented in `README.md` for any manual imports via phpMyAdmin or MySQL CLI.
3. Call stored procedures from PHP PDO backend (`$pdo->prepare("CALL sp_record_delivery_payment(?, ?, ?, ?, ?)")`).

---
**Related Notes:**
- [[School & Computer Science (Map of Content)]]
- [[PMS Inventory System - Database Design & Implementation Plan]]
- [[PMS Inventory System - Project Architecture & Requirements Proposal]]
