# 📊 Excel & Pandas Cheat Sheet
## คู่มือย่อ — การจัดการ Excel ด้วย Python

---

## 📖 Reading Excel Files

```python
import pandas as pd

# อ่านไฟล์ Excel
df = pd.read_excel('file.xlsx')
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')
df = pd.read_excel('file.xlsx', sheet_name=None)  # ทุก sheet → dict

# อ่าน CSV
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv', encoding='utf-8-sig')  # ภาษาไทย
```

## 🔍 Data Inspection

```python
df.head()              # 5 แถวแรก
df.tail(10)            # 10 แถวท้าย
df.info()              # โครงสร้างข้อมูล
df.describe()          # สถิติ (mean, std, min, max)
df.shape               # (rows, columns)
df.columns             # ชื่อคอลัมน์
df.dtypes              # ชนิดข้อมูล
df.isnull().sum()      # นับ null ในแต่ละคอลัมน์
len(df)                # จำนวนแถว
```

## 🎯 Data Selection

```python
# เลือกคอลัมน์
df['col']                       # 1 คอลัมน์ → Series
df[['col1', 'col2']]           # หลายคอลัมน์ → DataFrame

# เลือกแถว (by index)
df.iloc[0]                      # แถวแรก
df.iloc[0:5]                    # แถว 0-4
df.iloc[0:5, 0:3]              # แถว 0-4, คอลัมน์ 0-2

# เลือกแถว (by label)
df.loc[0]
df.loc[0:5, 'col_name']
df.loc[0:5, ['col1', 'col2']]
```

## 🔎 Filtering

```python
# เงื่อนไขเดียว
df[df['age'] > 30]
df[df['status'] == 'completed']

# หลายเงื่อนไข
df[(df['age'] > 30) & (df['city'] == 'Bangkok')]     # AND
df[(df['age'] > 30) | (df['city'] == 'Bangkok')]     # OR
df[~(df['status'] == 'failed')]                       # NOT

# IN / BETWEEN
df[df['name'].isin(['Alice', 'Bob'])]
df[df['value'].between(1.0, 5.0)]

# String filtering
df[df['name'].str.contains('Smith')]
df[df['name'].str.startswith('Dr')]
```

## 🧹 Data Cleaning

```python
# จัดการ Null
df.dropna()                         # ลบแถวที่มี null
df.dropna(subset=['col1'])          # ลบเฉพาะ null ใน col1
df.fillna(0)                        # แทน null ด้วย 0
df['col'].fillna(df['col'].mean())  # แทนด้วยค่าเฉลี่ย

# ลบ duplicates
df.drop_duplicates()
df.drop_duplicates(subset=['col1', 'col2'])

# แทนค่า
df['col'].replace('old', 'new')
df['col'].replace({'ND': None, 'N/D': None, '<LOD': None})

# String operations
df['col'].str.strip()           # ลบ whitespace
df['col'].str.upper()
df['col'].str.lower()

# Type conversion
df['col'] = df['col'].astype(float)
df['date'] = pd.to_datetime(df['date'])
```

## 📊 Grouping & Aggregation

```python
# Group By
df.groupby('lab')['concentration'].mean()
df.groupby('lab').agg({'As': 'mean', 'Pb': 'max'})

# Pivot Table
df.pivot_table(values='concentration', index='lab', columns='element', aggfunc='mean')

# Value Counts
df['status'].value_counts()

# Multiple aggregations
df.groupby('lab').agg(
    count=('sample_id', 'count'),
    avg_as=('As', 'mean'),
    max_pb=('Pb', 'max'),
).round(3)
```

## 📝 Writing Excel Files

```python
# เขียน Excel
df.to_excel('output.xlsx', index=False)
df.to_excel('output.xlsx', sheet_name='Results', index=False)

# เขียนหลาย sheets
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sheet1', index=False)
    df2.to_excel(writer, sheet_name='Sheet2', index=False)

# เขียน CSV
df.to_csv('output.csv', index=False, encoding='utf-8-sig')
```

## 🎨 openpyxl Formatting

```python
from openpyxl import Workbook, load_workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

wb = load_workbook('file.xlsx')
ws = wb.active

# อ่านค่า
value = ws['A1'].value
value = ws.cell(row=1, column=1).value

# เขียนค่า
ws['A1'] = 'Hello'
ws.cell(row=1, column=1, value='Hello')

# Formatting
ws['A1'].font = Font(bold=True, size=14, color='FF0000')
ws['A1'].fill = PatternFill(start_color='FFFF00', fill_type='solid')
ws['A1'].alignment = Alignment(horizontal='center', vertical='center')

# Column width
ws.column_dimensions['A'].width = 20

# Save
wb.save('output.xlsx')
```

## 📋 Sorting

```python
df.sort_values('col')                          # เรียงน้อยไปมาก
df.sort_values('col', ascending=False)         # มากไปน้อย
df.sort_values(['col1', 'col2'])               # เรียงหลายคอลัมน์
```

## ➕ Adding/Modifying Columns

```python
df['new_col'] = df['col1'] + df['col2']       # คำนวณ
df['total'] = df[['As', 'Pb', 'Cd']].sum(axis=1)
df['risk'] = df['total'].apply(lambda x: 'HIGH' if x > 10 else 'LOW')
```

---

### 📌 Quick Reference — pd.read_excel() Parameters
| Parameter | Description | Example |
|-----------|-------------|---------|
| `sheet_name` | Sheet ที่จะอ่าน | `'Sheet1'`, `0`, `None` |
| `header` | แถวที่เป็น header | `0` (default), `None` |
| `skiprows` | ข้ามกี่แถวแรก | `3` |
| `usecols` | คอลัมน์ที่จะอ่าน | `'A:C'`, `[0,1,2]` |
| `dtype` | กำหนดชนิดข้อมูล | `{'col': str}` |
