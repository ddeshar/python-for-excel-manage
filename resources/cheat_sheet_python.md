# 🐍 Python Basics Cheat Sheet
## คู่มือย่อ — Python พื้นฐาน

---

## 📌 Variables & Data Types

```python
# ตัวแปร (ไม่ต้องประกาศชนิด)
name = "Alice"          # str (ข้อความ)
age = 25                # int (จำนวนเต็ม)
height = 1.75           # float (ทศนิยม)
is_active = True        # bool (True/False)
data = None             # NoneType (ว่าง)

# ตรวจชนิด
type(name)              # <class 'str'>
isinstance(age, int)    # True
```

## 📋 Collections (ที่เก็บข้อมูลหลายค่า)

```python
# List (เปลี่ยนค่าได้, มีลำดับ)
elements = ["As", "Pb", "Cd", "Hg"]
elements[0]             # "As" (index เริ่มจาก 0)
elements.append("Cu")   # เพิ่มท้าย
elements.remove("Cu")   # ลบ
len(elements)           # จำนวนสมาชิก

# Dictionary (key-value pairs)
element = {
    "symbol": "As",
    "name": "Arsenic",
    "wavelength": 188.979
}
element["symbol"]        # "As"
element.get("unit", "mg/kg")  # default ถ้าไม่มี key
element.keys()           # dict_keys(['symbol', 'name', 'wavelength'])
element.values()         # dict_values(['As', 'Arsenic', 188.979])

# Tuple (เปลี่ยนค่าไม่ได้)
point = (10, 20)

# Set (ไม่ซ้ำ, ไม่มีลำดับ)
unique = {"As", "Pb", "As"}  # → {"As", "Pb"}
```

## 🔄 Control Flow

```python
# if / elif / else
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
else:
    grade = "C"

# for loop
for element in ["As", "Pb", "Cd"]:
    print(element)

for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i, elem in enumerate(elements):  # index + value
    print(f"{i}: {elem}")

# while loop
count = 0
while count < 5:
    count += 1

# List Comprehension (for loop แบบย่อ)
squares = [x**2 for x in range(5)]             # [0, 1, 4, 9, 16]
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]
```

## 🔧 Functions

```python
# สร้างฟังก์ชัน
def calculate_average(values):
    """คำนวณค่าเฉลี่ย"""
    return sum(values) / len(values)

result = calculate_average([1, 2, 3, 4, 5])  # 3.0

# Default parameter
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")           # "Hello, Alice!"
greet("Bob", "Hi")       # "Hi, Bob!"

# Lambda (ฟังก์ชันย่อ)
double = lambda x: x * 2
double(5)                # 10
```

## 📝 String Operations

```python
# f-string (แนะนำ)
name = "Alice"
f"Hello, {name}!"               # "Hello, Alice!"
f"Pi = {3.14159:.2f}"           # "Pi = 3.14"
f"Count: {42:05d}"              # "Count: 00042"

# String methods
text = "  Hello World  "
text.strip()                     # "Hello World"
text.upper()                     # "  HELLO WORLD  "
text.lower()                     # "  hello world  "
text.replace("World", "Python")  # "  Hello Python  "
text.split()                     # ["Hello", "World"]
",".join(["a", "b", "c"])       # "a,b,c"

# Check
"Hello" in text                  # True
text.startswith("  Hello")       # True
text.endswith("  ")              # True
```

## 📁 File Operations

```python
# เขียนไฟล์
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("Hello\n")
    f.write("World\n")

# อ่านไฟล์
with open("output.txt", "r", encoding="utf-8") as f:
    content = f.read()          # ทั้งไฟล์
    
with open("output.txt", "r") as f:
    lines = f.readlines()       # list ของแต่ละบรรทัด

# ตรวจสอบไฟล์
import os
os.path.exists("file.txt")      # True/False
os.path.isfile("file.txt")      # True ถ้าเป็นไฟล์
os.path.isdir("folder")         # True ถ้าเป็น folder
os.makedirs("new_dir", exist_ok=True)  # สร้าง folder
```

## ⚠️ Error Handling

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
except Exception as e:
    print(f"Error: {e}")
finally:
    print("This always runs")

# ใช้บ่อยกับ database/file
try:
    df = pd.read_excel("file.xlsx")
except FileNotFoundError:
    print("File not found!")
except Exception as e:
    print(f"Error reading file: {e}")
```

## 📦 Useful Built-in Functions

```python
# Math
abs(-5)                  # 5
round(3.14159, 2)       # 3.14
max(1, 2, 3)            # 3
min(1, 2, 3)            # 1
sum([1, 2, 3, 4])       # 10

# Type conversion
int("42")               # 42
float("3.14")           # 3.14
str(42)                 # "42"
list(range(5))          # [0, 1, 2, 3, 4]
dict(a=1, b=2)          # {"a": 1, "b": 2}

# Sorting
sorted([3, 1, 2])             # [1, 2, 3]
sorted(items, key=len)        # sort by length
sorted(items, reverse=True)   # descending

# zip (รวม list)
names = ["As", "Pb"]
values = [0.5, 1.2]
for name, val in zip(names, values):
    print(f"{name}: {val}")
```

## 🌐 Environment Variables

```python
import os
from dotenv import load_dotenv

# โหลดจาก .env file
load_dotenv()

# อ่านค่า
db_host = os.getenv("DB_HOST", "localhost")  # default = "localhost"
db_port = int(os.getenv("DB_PORT", "5432"))

# ⚠️ ห้ามเขียน password ลงในโค้ดตรงๆ!
```

## 📅 Date & Time

```python
from datetime import datetime, timedelta

now = datetime.now()
today = datetime.now().date()
now.strftime("%Y-%m-%d %H:%M:%S")    # "2026-01-15 14:30:00"
now.isoformat()                        # "2026-01-15T14:30:00"

# Parse
dt = datetime.strptime("2026-01-15", "%Y-%m-%d")

# Math
tomorrow = today + timedelta(days=1)
```

---

### 📌 Quick Reference — Common Format Specifiers
| Format | Meaning | Example |
|--------|---------|---------|
| `%Y` | Year (4 digits) | 2026 |
| `%m` | Month (01-12) | 01 |
| `%d` | Day (01-31) | 15 |
| `%H` | Hour (00-23) | 14 |
| `%M` | Minute (00-59) | 30 |
| `%S` | Second (00-59) | 00 |
