# 🧹 Sales Data Cleaning Project (Excel)

End-to-end documentation of how a raw, messy retail sales dataset (12,000 orders) was cleaned into an analysis-ready dataset (11,955 orders).

| | |
|---|---|
| **Domain** | Retail / E-commerce sales (India) |
| **Tool** | Microsoft Excel |
| **Raw file** | `Raw_data.xlsx` (sheet: `Raw_Data`) |
| **Cleaned file** | `Cleaned_data.xlsx` (sheet: `Sheet1`) |
| **Period covered** | Jan 2024 – Aug 2026 (order dates) |

---

## 📌 1. Objective

Raw data had duplicates, inconsistent text, invalid dates, broken discount values, wrong sales figures and junk phone numbers. The goal was to produce a **reliable, consistent dataset** that can be used directly for MIS reports, SQL analysis and Power BI dashboards.

---

## 📂 2. Dataset Overview

| Metric | Raw | Cleaned |
|---|---|---|
| Rows | 12,000 | **11,955** |
| Columns | 25 | 25 |
| Duplicate Order_IDs | 45 | **0** |
| Missing `Sales` | 28 | **0** |
| Missing `Discount` | 24 | **0** |
| Invalid `Order_Date` | 25 | 25 (set blank – see Limitations) |
| Missing `Delivery_Date` | 1,711 | 0 (see Known Issues ⚠️) |

### Column Dictionary

| Column | Description |
|---|---|
| `Order_ID` | Unique order identifier (`ORD-1xxxxx`) |
| `Order_Date` / `Delivery_Date` | Date of order / delivery |
| `Customer` *(was `Customer_Name`)* | Customer name |
| `Gender`, `Age` | Customer demographics (Age: 18–60) |
| `Region`, `State`, `City` | Location |
| `Category`, `Product` | Product details (4 categories) |
| `Quantity`, `Unit_Price` | Units ordered and price per unit |
| `Discount` | Discount as a decimal (0 – 0.25) |
| `Sales` | `Quantity × Unit_Price × (1 − Discount)` |
| `Cost`, `Profit` | Cost and `Sales − Cost` |
| `Payment_Mode`, `Shipping_Mode` | Payment / shipping method |
| `Order_Status` | Delivered, Pending, Returned, Cancelled |
| `Salesperson`, `Customer_Rating`, `Returned` | Sales rep, rating (1–5), return flag |
| `Email`, `Phone` | Contact details |

---

## 🔍 3. Data Quality Issues Found (Raw Data)

| # | Issue | Records affected | Example |
|---|---|---|---|
| 1 | Duplicate rows (same `Order_ID`) | 45 | Exact copies of earlier rows |
| 2 | Inconsistent `Region` text (case / spaces) | multiple | `' North '`, `NORTH`, `north` |
| 3 | Inconsistent `Payment_Mode` text | multiple | `Credit card`, `credit Card`, `UPI `, `upi` |
| 4 | Extra spaces / casing in customer names | 199 names changed | `'  Suhel Ali  '` |
| 5 | Invalid `Order_Date` (text, impossible dates) | 25 | `32/04/2022`, `31/02/2025`, `45/13/2024` |
| 6 | Invalid `Discount` values | 46 | `10%` (19), `1.2` (16), `-0.1` (11) |
| 7 | Missing `Discount` | 24 | blank |
| 8 | Missing `Sales` | 28 | blank |
| 9 | Wrong `Sales` (does not match Qty × Price × (1−Disc)) | 31 | `-5000` (10 rows), `45000` placeholder values |
| 10 | Invalid `Phone` numbers | 70 | `12345` (26), `987654` (24), `98ABC12345` (20) |
| 11 | Spaces inside `Email` | 81 | `vikas.verma @company.com` |
| 12 | Missing `Delivery_Date` | 1,711 | Mostly Cancelled orders (1,667) |

---

## 🛠️ 4. Cleaning Steps Performed

1. **Removed duplicates** – 45 duplicate rows dropped (first occurrence kept) → 11,955 unique orders.
2. **Standardised text** – trimmed spaces and applied proper case on `Region`, `Payment_Mode` and customer names.
   - `Region` → 4 clean values: East, North, South, West
   - `Payment_Mode` → 5 clean values: Cash, Credit Card, Debit Card, Net Banking, Upi
3. **Renamed column** – `Customer_Name` → `Customer`.
4. **Fixed `Order_Date`** – converted to proper date format; impossible dates (25 rows) could not be recovered and were left blank.
5. **Fixed `Discount`** – converted `10%` → decimal, and replaced invalid (`-0.1`, `1.2`) and missing values by back-calculating from `Sales ÷ (Quantity × Unit_Price)`. All discounts now lie in the valid set {0, 0.05, 0.10, 0.15, 0.20, 0.25}.
6. **Fixed `Sales`** – recomputed as `Quantity × Unit_Price × (1 − Discount)` for the 28 missing and 31 incorrect rows.
7. **Validated `Profit`** – confirmed `Profit = Sales − Cost` holds for all 11,955 rows after the Sales fix (0 mismatches).
8. **Cleaned `Email`** – removed stray spaces; all emails now follow a valid format (0 invalid).
9. **Cleaned `Phone`** – kept valid 10-digit numbers; anything non-numeric or not 10 digits was flagged as `Invalid`.
10. **Filled `Delivery_Date`** – blanks were filled (see Known Issues).

---

## ✅ 5. Validation Checks (After Cleaning)

| Check | Result |
|---|---|
| Duplicate `Order_ID` | 0 ✅ |
| Same Order_ID set as raw (after dedup) | Yes ✅ |
| `Sales = Qty × Price × (1 − Discount)` | 0 mismatches ✅ |
| `Profit = Sales − Cost` | 0 mismatches ✅ |
| Missing `Sales` / `Discount` | 0 ✅ |
| Invalid email format | 0 ✅ |
| `Region` / `Payment_Mode` unique values | 4 / 5 ✅ |
| Age range | 18 – 60 ✅ |
| `Quantity` range | 1 – 6 ✅ |
| `Customer_Rating` range | 1 – 5 ✅ |

### Impact on key totals

| Metric | Raw (after removing duplicates) | Cleaned |
|---|---|---|
| Total Sales | ₹41.27 Cr | ₹41.39 Cr |
| Total Profit | – | ₹5.96 Cr |

Sales rose slightly because missing and negative values were corrected.

---

## ⚠️ 6. Known Issues & Limitations

These are things I found while comparing the two files. Fix them before using this dataset for final reporting.

1. **`Delivery_Date` was filled incorrectly.** All 1,702 blanks were filled, including **1,667 Cancelled** and 7 Pending orders, which should never have a delivery date. As a result, **829 rows now have `Delivery_Date` earlier than `Order_Date`** (raw data had 0 such rows). The values look copied from neighbouring rows (e.g. `ORD-100001`, Cancelled, shows the delivery date of `ORD-100002`).
   **Recommended fix:** restore blanks for Cancelled/Pending orders; for the 20 Delivered and 8 Returned rows with a missing date, leave them blank or use `Order_Date + median shipping days` (about 4 days).
2. **25 rows have blank `Order_Date`.** Original values were impossible dates, so they cannot be used in time-series analysis. Either exclude them from date-based reports or confirm the real dates with the data source.
3. **`Region` and `State` do not always agree.** After standardising, many states map to multiple regions (e.g. Delhi appears under both North and South). Text was cleaned but this logical mismatch was **not** fixed. Consider deriving `Region` from `State` using one mapping table.
4. **Floating-point noise in `Discount`.** Back-calculated values like `0.1499999` should be rounded to 2 decimals.
5. **`Phone` has mixed types.** Valid numbers are stored as numbers while 70 invalid ones are the text `Invalid`. Use text for the whole column (or blank for invalid) for cleaner SQL / Power BI imports.
6. **1,848 orders have negative profit** (`Cost > Sales`). These were kept because they are valid records, not errors, and are worth a separate analysis (heavy discounts on high-cost items).
7. **Gender vs name mismatches** (e.g. a female-sounding name tagged Male) exist in the data. They cannot be verified, so they were left unchanged.

---

## 💡 7. Key Learnings

- Always recompute derived columns (`Sales`, `Profit`) instead of trusting stored values.
- Validate *relationships* between columns (Order date ≤ Delivery date, State → Region), not just individual columns.
- Do not fill a missing value just to get a "0 nulls" result. A blank can be the correct value (e.g. no delivery for a cancelled order).

---

## 🚀 8. Next Steps

- [ ] Fix `Delivery_Date` logic and rerun validation
- [ ] Create `State → Region` mapping table
- [ ] Round `Discount` to 2 decimals
- [ ] Load cleaned data into **PostgreSQL** for SQL analysis
- [ ] Build **Power BI** dashboard: sales by region/category, profit trends, return rate, salesperson performance

---

## 📁 Repository Structure

```
Sales-Data-Cleaning-Excel/
├── Raw_data.xlsx        # Original messy dataset (12,000 rows)
├── Cleaned_data.xlsx    # Cleaned dataset (11,955 rows)
└── README.md            # Project documentation
```

---

## 👤 Author

**Anash** – BCA graduate | Aspiring Data Analyst
Skills: MS Excel (advanced), SQL / PostgreSQL, Power BI, Python (learning)

🔗 GitHub: [AnashGulfamAli](https://github.com/AnashGulfamAli)
