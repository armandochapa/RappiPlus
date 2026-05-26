# 📊 RappiPlus: From Data to Business Decisions

> **End-to-end data analytics project** | Python · SQL · Power BI  
> Data quality assessment, profitability analysis, funnel optimization, cohort retention, A/B testing, and executive dashboard

---

## 🎯 Project Overview

This project delivers a full-cycle business analytics solution for **RappiPlus**, a subscription-based delivery service operating in **Mexico, Argentina, and Colombia**. Starting from raw transactional data, the analysis progresses through six structured stages — from data validation to an interactive executive dashboard — to provide actionable insights that support strategic decision-making.

### Business Questions Addressed

- Can we trust the data we have? *(Data quality)*
- Is the business profitable? *(Revenue, costs & profit analysis)*
- Where are users dropping off? *(Conversion funnel)*
- Are users coming back? *(Cohort retention)*
- Does the new checkout design actually convert better? *(A/B testing)*
- How can results be communicated to stakeholders? *(Power BI dashboard)*

---

## 🗂️ Project Architecture

```
rappiplus-analysis/
│
├── 📓 S12_Estudiante_Proyecto_Final.ipynb   ← Full Python + SQL analysis
├── 📊 Proyecto_final_Armando.pbix           ← Power BI executive dashboard
└── 📄 README.md
```

---

## 📦 Datasets

| Dataset | Source | Description |
|---------|--------|-------------|
| `rappiplus_orders_raw.csv` | CSV | Orders, quantities, unit prices, discounts, revenue |
| `rappiplus_catalog.csv` | CSV | Product catalog with unit costs, categories, and suppliers |
| `rappiplus_marketing_spend.csv` | CSV | Marketing investment by channel, country, and date |
| `events` | PostgreSQL | User event tracking (visits, cart, checkout, purchase) |
| `users` | PostgreSQL | User registration data for cohort construction |
| `user_activity` | PostgreSQL | Daily activity flags by user for retention analysis |
| `experiment_checkout_ui.csv` | CSV | A/B experiment results for checkout UI redesign |

---

## 🔄 Analysis Pipeline

### Step 1 · Data Quality & Cleaning

Validated and cleaned all three CSV datasets before analysis:

**`orders`**
- Converted `fecha_hora_pedido` to proper `datetime` format
- Removed records with invalid quantities (`cantidad ≤ 0`) and amounts (`monto_total ≤ 0`)
- Verified amount consistency: `monto_total ≈ (cantidad × precio_unitario) − monto_descuento` (±$0.02 tolerance)
- Eliminated duplicate `id_pedido` entries, keeping first occurrence
- Standardized country names using `.str.title()` to resolve capitalization inconsistencies
- Retained records with partial nulls (product/country) when their revenue data remained valid

**`catalog`**
- Confirmed 100% data integrity — no invalid costs, no duplicates, no null values
- 7 unique products across 3 categories (Electronics, Home, Fashion)

**`marketing`**
- Removed 66 duplicate records by `(fecha, pais, canal)` composite key
- Verified all spend values are positive
- Confirmed date range: Jan–Jun 2025 across 3 countries and 3 channels
- Final dataset: 1,554 records · $2,754,632.08 total spend

---

### Step 2 · Business Profitability Analysis

Merged `orders_clean` with `catalog_clean` to compute product-level costs, then joined with `marketing_final` for full P&L:

| KPI | Formula |
|-----|---------|
| **Revenue** | `SUM(monto_total)` |
| **COGS** | `SUM(cantidad × costo_unitario)` |
| **Gross Profit** | `Revenue − COGS` |
| **Net Profit** | `Gross Profit − Marketing Spend` |
| **Gross Margin** | `Gross Profit / Revenue` |
| **Net Margin** | `Net Profit / Revenue` |
| **Average Order Value** | `MEAN(monto_total)` |
| **Avg Items per Order** | `MEAN(cantidad)` |

Additional breakdowns: top products by units sold, marketing spend by channel (`paid_search`, `organic`, `social`) and by country.

---

### Step 3 · Conversion Funnel Analysis (SQL)

Queried the `events` table in PostgreSQL to map user drop-off across the full purchase journey:

```sql
first_visit → add_to_cart → select_item → begin_checkout → add_payment_info → purchase
```

Used `COUNT(DISTINCT id_usuario)` per stage and `LAG()` window function to compute step-by-step conversion rates and identify the highest drop-off point.

---

### Step 4 · Cohort Retention Analysis (SQL)

Built monthly cohorts from the `users` table and tracked weekly retention using `user_activity`:

- **Cohort definition:** Month of `fecha_registro`
- **Retention windows:** Week 1 (days 1–7), Week 2 (days 8–14), Week 3 (days 15–21)
- **Metric:** `retained_users / cohort_initial_users × 100`

Used `DATE_TRUNC`, `JOIN`, `CASE WHEN`, and `COALESCE` to build a pivot-style cohort retention table.

---

### Step 5 · A/B Statistical Test

Evaluated whether the new checkout UI redesign significantly improved purchase conversion.

**Experiment setup:**
- **Control:** Original checkout page
- **Treatment:** Redesigned checkout UI
- **Target metric:** `convirtio` (1 = purchase completed, 0 = not)
- **Significance level:** α = 0.05

**Statistical tests applied:**

| Test | Statistic | p-value | Conclusion |
|------|-----------|---------|------------|
| Z-test of Proportions | z | p | Significant / Not significant |
| Chi-Square Test | χ² | p | Consistent with Z-test |

> Note: Both tests are mathematically equivalent for binary outcomes: χ² = z². Results are consistent across methods.

---

### Step 6 · Executive Dashboard (Power BI)

An interactive **3-page Power BI report** built on a star schema model integrating orders, catalog, and date tables.

#### Page 1 · Executive Overview
| Element | Type | Details |
|---------|------|---------|
| Revenue Total | KPI Card | Total accumulated revenue |
| Profit Total | KPI Card | Total accumulated profit |
| Marketing Spend | KPI Card | Total marketing investment |
| Monthly Revenue Trend | Line Chart | Revenue evolution by Año-Mes |
| Monthly Profit Trend | Line Chart | Profit evolution by Año-Mes |
| YTD Accumulated | Combo Chart | Revenue YTD + Profit YTD vs. monthly |
| Revenue & Profit by Category | Clustered Column | Side-by-side comparison |
| Filters | Slicers | Country · Product Category · Date range |

#### Page 2 · Product Detail (Drill-through)
| Element | Type | Details |
|---------|------|---------|
| Revenue by Product | Bar Chart | Horizontal ranking by total revenue |
| Product Detail Table | Table | Product · Revenue · Unit Cost · Profit · Category |
| Filters | Slicers | Date range · Product Category |

#### Page 3 · Orders
| Element | Type | Details |
|---------|------|---------|
| Monthly Revenue | Line Chart | Revenue trend by period |
| Product Table | Table | Category and product name breakdown |

#### DAX Measures

```dax
Revenue Total      = SUM(orders[monto_total])
Profit Total       = [Revenue Total] - [COGS Total]
Marketing Spend    = SUM(marketing[gasto])
Revenue YTD        = TOTALYTD([Revenue Total], dates[Date])
Profit YTD         = TOTALYTD([Profit Total], dates[Date])
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Python 3** | Data wrangling, EDA, statistical testing |
| **pandas / numpy** | Data manipulation and numerical computation |
| **scipy · statsmodels** | Z-test of proportions, Chi-square test |
| **matplotlib · seaborn** | Data visualization |
| **PostgreSQL / SQLAlchemy** | Funnel and cohort queries |
| **Power BI Desktop / Cloud** | Interactive executive dashboard |
| **DAX** | KPI measures and YTD calculations |
| **Power Query (M)** | Data transformation in Power BI |

---

## 🚀 How to Run

### Python Notebook

**Option A — Google Colab (Recommended)**
1. Open [Google Colab](https://colab.research.google.com/)
2. Upload `S12_Estudiante_Proyecto_Final.ipynb`
3. Run cells sequentially — CSV files are loaded directly from hosted URLs
4. For SQL steps, a valid database connection is required

**Option B — Local Jupyter**
```bash
pip install pandas numpy matplotlib seaborn scipy statsmodels sqlalchemy psycopg2 jupyter
jupyter notebook S12_Estudiante_Proyecto_Final.ipynb
```

### Power BI Dashboard

1. Download [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) (free)
2. Open `Proyecto_final_Armando.pbix`
3. Navigate pages using slicers and drill-through interactions
4. To publish: upload to [app.powerbi.com](https://app.powerbi.com) → Import → Local file

---

## 📋 Requirements

```
Python 3.8+
pandas
numpy
matplotlib
seaborn
scipy
statsmodels
sqlalchemy
psycopg2-binary
jupyter
```

---

## 📈 Key Deliverables

- ✅ Three production-ready cleaned datasets exported as `.csv`
- ✅ Full P&L breakdown with gross and net margin calculations
- ✅ Step-by-step conversion funnel with drop-off identification
- ✅ Monthly cohort retention matrix (weeks 1–3)
- ✅ Statistically validated A/B test conclusion with dual-method verification
- ✅ Interactive Power BI dashboard with drill-through, YTD metrics, and multi-country slicing

---

## 🤝 Contributing

This project is part of my professional data analytics portfolio. If you have suggestions or feedback, feel free to open an issue or submit a pull request.

---

**Author:** Armando Chapa  
**Date:** May 2026  
**Version:** 1.0  
**Tools:** Python 3 · PostgreSQL · Power BI Cloud 2026.04
