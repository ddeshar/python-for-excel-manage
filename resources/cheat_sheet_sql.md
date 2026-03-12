# 🗄️ SQL & PostgreSQL Cheat Sheet
## คู่มือย่อ — SQL สำหรับการจัดการข้อมูล

---

## 📖 Basic Queries

```sql
-- เลือกทั้งหมด
SELECT * FROM elements;

-- เลือกบางคอลัมน์
SELECT symbol, name, wavelength FROM elements;

-- กรอง
SELECT * FROM test_orders WHERE status = 'completed';

-- หลายเงื่อนไข
SELECT * FROM test_orders 
WHERE status = 'completed' AND analyst_name = 'Dr. Smith';

-- OR
SELECT * FROM test_orders 
WHERE status = 'pending' OR status = 'in_progress';

-- IN
SELECT * FROM test_orders 
WHERE status IN ('pending', 'in_progress');

-- BETWEEN
SELECT * FROM test_results 
WHERE concentration BETWEEN 1.0 AND 5.0;

-- LIKE (ค้นหาคำ)
SELECT * FROM test_orders WHERE order_id LIKE '%2026%';
SELECT * FROM elements WHERE name LIKE 'A%';     -- ขึ้นต้นด้วย A

-- NULL
SELECT * FROM test_results WHERE concentration IS NULL;
SELECT * FROM test_results WHERE concentration IS NOT NULL;
```

## 📋 Sorting & Limiting

```sql
-- เรียงลำดับ
SELECT * FROM test_orders ORDER BY order_date;          -- น้อย→มาก
SELECT * FROM test_orders ORDER BY order_date DESC;     -- มาก→น้อย
SELECT * FROM test_orders ORDER BY lab, order_date DESC; -- หลายคอลัมน์

-- จำกัดจำนวน
SELECT * FROM test_orders LIMIT 10;
SELECT * FROM test_orders LIMIT 10 OFFSET 20;          -- ข้าม 20 แถวแรก
```

## 📊 Aggregate Functions

```sql
-- นับ
SELECT COUNT(*) FROM test_orders;
SELECT COUNT(DISTINCT analyst_name) FROM test_orders;

-- ค่าเฉลี่ย, ผลรวม, min, max
SELECT 
    AVG(concentration) AS avg_conc,
    SUM(concentration) AS total_conc,
    MIN(concentration) AS min_conc,
    MAX(concentration) AS max_conc
FROM test_results;

-- ปัดทศนิยม
SELECT ROUND(AVG(concentration), 3) AS avg_conc FROM test_results;
```

## 📦 GROUP BY & HAVING

```sql
-- จัดกลุ่มและนับ
SELECT status, COUNT(*) AS order_count
FROM test_orders
GROUP BY status
ORDER BY order_count DESC;

-- Aggregate ตามกลุ่ม
SELECT 
    e.symbol,
    COUNT(*) AS measurement_count,
    ROUND(AVG(r.concentration), 3) AS avg_conc,
    ROUND(MIN(r.concentration), 3) AS min_conc,
    ROUND(MAX(r.concentration), 3) AS max_conc
FROM test_results r
JOIN elements e ON r.element_id = e.id
GROUP BY e.symbol;

-- HAVING (กรองหลัง GROUP BY)
SELECT analyst_name, COUNT(*) AS cnt
FROM test_orders
GROUP BY analyst_name
HAVING COUNT(*) >= 5;
```

## 🔗 JOIN Operations

```sql
-- INNER JOIN (เฉพาะที่ match กัน)
SELECT o.order_id, l.code AS lab
FROM test_orders o
INNER JOIN laboratories l ON o.laboratory_id = l.id;

-- LEFT JOIN (ทุกแถวจากตารางซ้าย)
SELECT o.order_id, l.code AS lab
FROM test_orders o
LEFT JOIN laboratories l ON o.laboratory_id = l.id;

-- JOIN หลายตาราง
SELECT 
    o.order_id,
    l.code AS lab,
    e.symbol AS element,
    r.concentration
FROM test_results r
INNER JOIN test_orders o ON r.order_id = o.id
INNER JOIN elements e ON r.element_id = e.id
INNER JOIN laboratories l ON o.laboratory_id = l.id;
```

## 📐 Window Functions

```sql
-- ROW_NUMBER — ลำดับ
SELECT 
    order_id,
    concentration,
    ROW_NUMBER() OVER (ORDER BY concentration DESC) AS rank
FROM test_results;

-- PARTITION BY — ลำดับภายในกลุ่ม
SELECT 
    e.symbol,
    r.concentration,
    ROW_NUMBER() OVER (
        PARTITION BY e.symbol 
        ORDER BY r.concentration DESC
    ) AS rank_in_element
FROM test_results r
JOIN elements e ON r.element_id = e.id;

-- RANK vs DENSE_RANK
-- RANK: 1, 2, 2, 4  (ข้ามอันดับ)
-- DENSE_RANK: 1, 2, 2, 3  (ไม่ข้าม)

-- Running Average
SELECT 
    order_date,
    concentration,
    AVG(concentration) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM test_results;

-- LAG / LEAD (ค่าก่อนหน้า / ถัดไป)
SELECT 
    order_date,
    concentration,
    LAG(concentration) OVER (ORDER BY order_date) AS prev_value,
    LEAD(concentration) OVER (ORDER BY order_date) AS next_value
FROM test_results;
```

## 📝 CTE (Common Table Expression)

```sql
-- CTE — อ่านง่ายกว่า Subquery
WITH avg_by_element AS (
    SELECT 
        element_id,
        AVG(concentration) AS avg_conc
    FROM test_results
    GROUP BY element_id
)
SELECT 
    e.symbol,
    ROUND(a.avg_conc, 3) AS avg_concentration,
    COUNT(*) AS above_avg_count
FROM test_results r
JOIN avg_by_element a ON r.element_id = a.element_id
JOIN elements e ON r.element_id = e.id
WHERE r.concentration > a.avg_conc
GROUP BY e.symbol, a.avg_conc;
```

## ✏️ Data Modification

```sql
-- INSERT
INSERT INTO elements (symbol, name, wavelength) 
VALUES ('As', 'Arsenic', 188.979);

-- INSERT หลายแถว
INSERT INTO elements (symbol, name, wavelength) VALUES
    ('As', 'Arsenic', 188.979),
    ('Pb', 'Lead', 220.353),
    ('Cd', 'Cadmium', 226.502);

-- UPDATE
UPDATE test_orders SET status = 'completed' WHERE order_id = 'ORD-001';

-- DELETE
DELETE FROM test_orders WHERE status = 'cancelled';
```

## 🏗️ Create Table

```sql
-- PostgreSQL
CREATE TABLE elements (
    id SERIAL PRIMARY KEY,
    symbol VARCHAR(10) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    wavelength FLOAT,
    unit VARCHAR(20) DEFAULT 'mg/kg',
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- สำหรับ Training (docker compose up -d)
-- Database: icp_training
-- User: icp_user / training_password
```

## 🐍 SQL with Python

```python
import pandas as pd
from sqlalchemy import create_engine, text

# สร้าง connection
engine = create_engine("postgresql://icp_user:training_password@localhost:5432/icp_training")

# อ่านข้อมูล → DataFrame
df = pd.read_sql("SELECT * FROM elements", engine)

# Parameterized query (ป้องกัน SQL Injection)
df = pd.read_sql(
    text("SELECT * FROM orders WHERE status = :s"),
    engine,
    params={"s": "completed"}
)

# เขียน DataFrame → Database
df.to_sql("table_name", engine, if_exists="replace", index=False)
# if_exists: 'fail' | 'replace' | 'append'

# Execute SQL
with engine.connect() as conn:
    conn.execute(text("INSERT INTO ... VALUES (...)"), {"param": value})
    conn.commit()
```

---

### 📌 Quick Reference — SQL Operator Priority
| Priority | Operator |
|----------|----------|
| 1 | `()` parentheses |
| 2 | `NOT` |
| 3 | `AND` |
| 4 | `OR` |

### 📌 Quick Reference — Common Date Functions (PostgreSQL)
```sql
NOW()                           -- เวลาปัจจุบัน
CURRENT_DATE                    -- วันที่ปัจจุบัน
DATE_TRUNC('month', date_col)   -- ตัดเป็นเดือน
EXTRACT(YEAR FROM date_col)     -- ดึงปี
AGE(date1, date2)               -- ความต่าง
```
