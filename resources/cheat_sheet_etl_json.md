# 🔄 ETL & JSON Cheat Sheet
## คู่มือย่อ — ETL Pipeline & JSON

---

## 📋 JSON Basics

### Python ↔ JSON

```python
import json

# Dict → JSON string
json_string = json.dumps(data, indent=2, ensure_ascii=False)

# JSON string → Dict
data = json.loads(json_string)

# Dict → JSON file
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)

# JSON file → Dict
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
```

### Python vs JSON Values
| Python | JSON |
|--------|------|
| `True` | `true` |
| `False` | `false` |
| `None` | `null` |
| `dict` | `object {}` |
| `list` | `array []` |
| `str` | `string ""` |
| `int/float` | `number` |

### pandas ↔ JSON

```python
# DataFrame → JSON
json_str = df.to_json(orient="records", date_format="iso")

# DataFrame → Dict
records = df.to_dict(orient="records")

# Dict/JSON → DataFrame
df = pd.DataFrame(records)
df = pd.read_json("data.json")
```

### orient Options for to_json()
| orient | รูปแบบ |
|--------|--------|
| `"records"` | `[{"col1": val, "col2": val}, ...]` |
| `"columns"` | `{"col1": {"0": val}, "col2": {"0": val}}` |
| `"index"` | `{"0": {"col1": val}, "1": {"col1": val}}` |
| `"values"` | `[[val1, val2], [val1, val2]]` |

---

## 🔄 ETL Pipeline Pattern

```
┌──────────┐    ┌──────────────┐    ┌──────────┐
│ EXTRACT  │ →  │  TRANSFORM   │ →  │   LOAD   │
│          │    │              │    │          │
│ อ่าน Excel│    │ Validate     │    │ Save DB  │
│ อ่าน CSV │    │ Clean        │    │ Save JSON│
│ อ่าน API │    │ Convert      │    │ Save File│
└──────────┘    └──────────────┘    └──────────┘
```

### Extract (อ่านข้อมูล)

```python
import pandas as pd

# จาก Excel
df = pd.read_excel("data.xlsx")
df = pd.read_excel("data.xlsx", sheet_name="Results")

# จาก CSV
df = pd.read_csv("data.csv")

# จาก Database
df = pd.read_sql("SELECT * FROM table", engine)

# จาก JSON
df = pd.read_json("data.json")
```

### Transform (แปลงข้อมูล)

```python
# กรอง
df_clean = df[df["status"] == "completed"].copy()

# ลบ null
df_clean = df.dropna(subset=["important_col"])

# แทนค่า
df_clean["value"] = df_clean["value"].replace({"ND": None, "<LOD": None})

# แปลงชนิด
df_clean["concentration"] = pd.to_numeric(df_clean["concentration"], errors="coerce")

# เพิ่มคอลัมน์
df_clean["total"] = df_clean[["As", "Pb", "Cd"]].sum(axis=1)

# Validate
assert len(df_clean) > 0, "No data after filtering!"
assert df_clean["concentration"].notna().all(), "Still has null values!"
```

### Load (บันทึก)

```python
# → Database
df.to_sql("table_name", engine, if_exists="append", index=False)

# → Excel
df.to_excel("output.xlsx", index=False)

# → JSON
with open("output.json", "w") as f:
    json.dump(records, f, indent=2)

# → CSV
df.to_csv("output.csv", index=False)
```

---

## 🔐 Database Connection

### SQLAlchemy (แนะนำ)

```python
from sqlalchemy import create_engine, text

# PostgreSQL (ใช้ Docker)
engine = create_engine("postgresql://icp_user:training_password@localhost:5432/icp_training")

# หรือจาก .env
from dotenv import load_dotenv
load_dotenv("../.env")
DATABASE_URL = f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('DB_NAME')}"
engine = create_engine(DATABASE_URL)

# อ่าน
df = pd.read_sql("SELECT * FROM table", engine)

# เขียน
df.to_sql("table", engine, if_exists="append", index=False)

# Execute
with engine.connect() as conn:
    conn.execute(text("INSERT INTO ..."), {"param": value})
    conn.commit()
```

### psycopg2 (PostgreSQL direct)

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost", port=5432,
    dbname="icp_insight",
    user="user", password="pass"
)
cur = conn.cursor()
cur.execute("SELECT * FROM elements")
rows = cur.fetchall()
conn.close()
```

### Environment Variables (ปลอดภัย)

```python
import os
from dotenv import load_dotenv

load_dotenv()  # อ่านจาก .env file

DATABASE_URL = f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('DB_NAME')}"
```

---

## 📝 Validation Patterns

### Filename Validation
```python
import re

def validate_filename(filename):
    errors = []
    
    # ตรวจ extension
    ext = os.path.splitext(filename)[1].lower()
    if ext not in (".xlsx", ".xls"):
        errors.append(f"Invalid file type: {ext}")
    
    # ตรวจ pattern
    lab = re.search(r"(L\d+)", filename, re.IGNORECASE)
    if not lab:
        errors.append("No lab code found")
    
    return {"valid": len(errors) == 0, "errors": errors}
```

### Data Validation
```python
def validate_data(df, required_columns, expected_count=None):
    errors = []
    
    # ตรวจคอลัมน์
    missing = set(required_columns) - set(df.columns)
    if missing:
        errors.append(f"Missing columns: {missing}")
    
    # ตรวจแถวว่าง
    if df.empty:
        errors.append("DataFrame is empty")
    
    # ตรวจจำนวน
    if expected_count and len(df) != expected_count:
        errors.append(f"Expected {expected_count} rows, got {len(df)}")
    
    return errors
```

---

## ⚠️ Error Handling in ETL

```python
def safe_etl(filepath, engine):
    """ETL with error handling"""
    try:
        # Extract
        df = pd.read_excel(filepath)
        
        # Transform
        df_clean = df.dropna()
        
        # Load
        df_clean.to_sql("results", engine, if_exists="append", index=False)
        
        return {"success": True, "rows": len(df_clean)}
        
    except FileNotFoundError:
        return {"success": False, "error": "File not found"}
    except pd.errors.EmptyDataError:
        return {"success": False, "error": "File is empty"}
    except Exception as e:
        return {"success": False, "error": str(e)}
```

---

## 🧪 Nested JSON (ข้อมูลซ้อนกัน)

### ICP Result JSON Structure
```json
{
    "order_id": "ORD-2026-0001",
    "laboratory": "L66",
    "panel": "EU4",
    "elements": ["As", "Pb", "Cd", "Hg"],
    "samples": [
        {
            "sample_id": "S01",
            "replicate": 1,
            "concentrations": {
                "As": 0.523,
                "Pb": 1.245,
                "Cd": 0.089,
                "Hg": 0.034
            }
        }
    ]
}
```

### Flatten Nested JSON
```python
# Nested → Flat
flat_rows = []
for sample in data["samples"]:
    for elem, conc in sample["concentrations"].items():
        flat_rows.append({
            "sample_id": sample["sample_id"],
            "element": elem,
            "concentration": conc,
        })
df_flat = pd.DataFrame(flat_rows)
```

### Build Nested JSON
```python
# Flat → Nested
result = {"elements": [], "samples": []}
for _, row in df.iterrows():
    sample = {
        "sample_id": row["sample_id"],
        "concentrations": {
            "As": float(row["As"]),
            "Pb": float(row["Pb"]),
        }
    }
    result["samples"].append(sample)
```

---

### 📌 Quick Reference — to_sql() Parameters
| Parameter | Description | Options |
|-----------|-------------|---------|
| `name` | ชื่อตาราง | `"table_name"` |
| `con` | Engine/Connection | `engine` |
| `if_exists` | ถ้าตารางมีอยู่แล้ว | `'fail'`, `'replace'`, `'append'` |
| `index` | เขียน index ด้วย | `True`, `False` |
| `dtype` | กำหนดชนิดคอลัมน์ | `{"col": sqlalchemy.String}` |
| `chunksize` | เขียนทีละกี่แถว | `1000` |
