<p align="center">
  <img src="assets/banner.svg" alt="Argos — E-Commerce Marketing Analysis"/>
</p>

# Argos E-Commerce Marketing Analysis — Corrected & Reproducible Edition

An end-to-end e-commerce business analysis covering January 2020 to June 2024. This corrected edition preserves the original raw data and project intent while repairing data-quality handling, cohort maturity logic, RFM segmentation reproducibility, funnel terminology, report consistency, and notebook portability.

## 1. Business Questions

The project addresses six questions:

1. Is revenue growing, stable, or declining?
2. Where does the purchase funnel lose the most sessions at each **specific stage**?
3. How do acquisition source and device differ in conversion and order value?
4. How quickly do purchasing customers return after their first order?
5. Which customer segments account for the largest share of historical revenue?
6. How do discount depth and category mix affect estimated merchandise gross margin?

## 2. Data Scope

- Analysis period: **2020-01 to 2024-06**
- Customers: **20,000**
- Sessions: **92,369**
- Events: **584,805**
- Completed orders: **25,716**
- Purchasing customers: **14,505**
- Products: **1,197**
- Completed-order revenue: **£2,711,160.44**

Raw source files remain unchanged under `data/raw/`.

## 3. Data-Quality Corrections

### 3.1 `order_items` relationship integrity

The raw `order_items.csv` contains **59,163 rows**, but **13,871 rows across 7,864 order IDs do not match the completed `orders` table**. Those unmatched rows are not treated as completed-order revenue or profit.

Corrected treatment:

- `data/outputs/order_items_clean.csv`: **45,292 item rows linked to completed orders**.
- `data/quality/order_items_unmatched.csv`: unmatched rows retained for audit rather than deleted.
- `data/quality/order_item_subtotal_reconciliation.csv`: order-level subtotal reconciliation.

### 3.2 Exact-looking line-item duplicates are preserved

There are 73 extra rows that look identical across the available item columns. The raw source has no `line_item_id`, and removing these rows breaks reconciliation to `orders.subtotal_gbp`. Therefore the corrected pipeline **does not automatically deduplicate them**.

Completed-order item subtotal: approximately **£2,917,451.25**  
Completed-order `orders.subtotal_gbp`: approximately **£2,917,448.10**  
Difference: **£3.15**, consistent with line-level rounding.

### 3.3 Revenue and profit are reconciled to the order ledger

For item/category analysis, each completed order's `total_gbp` is allocated across its item rows in proportion to each line's share of the order item subtotal. This guarantees that item-level allocated revenue reconciles to completed-order revenue.

- Completed-order revenue: **£2,711,160.44**
- Allocated item revenue: **£2,711,160.44**
- Estimated merchandise gross profit: **£828,539.36**
- Estimated merchandise gross margin: **30.56%**

The gross-profit metric is a merchandise estimate based on product cost and excludes fulfilment, marketing, tax, refunds, and other operating costs.

## 4. Analytical Corrections

### Cohort analysis

The original logic averaged immature cohorts as if unobserved future periods were zero. The corrected outputs:

- keep observed zeroes as zero;
- keep future/unobserved periods missing;
- include a cohort at M1/M3/M6/M12 only if that horizon is observable.

Maturity-adjusted cumulative repeat-purchase incidence:

| Checkpoint | Average | Eligible cohorts |
|---|---:|---:|
| M1 | 2.18% | 53 |
| M3 | 6.89% | 51 |
| M6 | 13.26% | 48 |
| M12 | 24.47% | 42 |

### RFM analysis

The corrected RFM implementation:

- uses a fixed analysis cutoff of **2024-06-30**;
- uses deterministic quintile ordering with `customer_id` as the tie-breaker;
- assigns score 5 to better Recency/Frequency/Monetary rank;
- evaluates `New Customers` before `Potential Loyalists`, making the former reachable;

Corrected segment summary:

| Segment | Customers | Customer share | Revenue share |
|---|---:|---:|---:|
| Loyal Customers | 3,653 | 25.18% | 38.13% |
| Lost | 6,016 | 41.48% | 24.12% |
| Champions | 1,182 | 8.15% | 17.14% |
| Cannot Lose Them | 669 | 4.61% | 9.66% |
| At Risk | 1,735 | 11.96% | 7.99% |
| Potential Loyalists | 1,036 | 7.14% | 2.63% |
| New Customers | 214 | 1.48% | 0.33% |

Champions + Loyal Customers account for **33.33% of purchasing customers and 55.27% of revenue**.

### Funnel terminology

The historical `cart_abandonment_pct` field is retained for compatibility, but it is explicitly documented as the **share of add-to-cart sessions that do not ultimately purchase**, which spans more than one stage.

The corrected outputs also expose precise stage metrics:

- Add-to-cart → checkout conversion: **54.93%**
- Add-to-cart → checkout drop-off: **45.07%**
- Checkout → purchase conversion: **74.73%**
- Checkout abandonment: **25.27%**
- Overall session → purchase CVR: **27.84%**

## 5. Corrected Business Findings

### Growth

The business is stable rather than trend-growing over the analysis window.

- First 12 months average revenue: approximately **£49,315/month**
- Last 12 months average revenue: approximately **£49,628/month**
- Change: **+0.64%**
- Monthly revenue linear-trend R²: approximately **0.002**

Order count and AOV are also broadly unchanged between the first and last 12 months.

### Channel and device

- Referral has the highest session purchase CVR at **28.64%**.
- Referral also has the highest maturity-adjusted M6 cumulative repeat-purchase rate at **14.42%**.
- Email has the lowest M6 rate at **11.39%**.
- Desktop, Tablet, and Mobile purchase CVR differ by only **0.17 percentage points**, so the data does not support a broad device-level conversion problem.

These are descriptive associations. The dataset contains no media cost or controlled experiment, so channel ROI and causal performance are not inferred.

### Customer value

- Overall repeat-purchase rate: **51.13%**.
- One-time purchasing customers: **7,089**.
- Cannot Lose Them contains **669** customers with approximately **£391.68 average historical revenue** and **1,026 days average recency**. This makes it a reasonable reactivation **test group**, not a proven highest-ROI segment.

### Discount and profit quality

- **57.59%** of completed orders have a discount.
- Estimated merchandise gross margin falls from **35.64% at 0% discount** to **19.24% at 20% discount**.
- 20% discount orders account for **14.29% of orders** but only **7.55% of estimated merchandise gross profit**.

This establishes margin compression, but the data does not prove whether discounts create enough incremental demand to offset it.

## 6. Project Structure

```text
Argos-Ecommerce-Marketing-Analysis-main/
├── README.md
├── requirements.txt
├── assets/
├── data/
│   ├── raw/                       # untouched source CSV files
│   ├── outputs/                   # regenerated corrected analytical outputs
│   └── quality/                   # audit, quarantine and reconciliation files
├── sql/
│   ├── 01_setup_and_base_tables.sql
│   ├── 02_funnel.sql
│   ├── 03_cohort.sql
│   ├── 04_rfm.sql
│   └── 05_profitability.sql
├── notebooks/
│   ├── 01_etl_and_setup.ipynb
│   ├── 02_funnel_analysis.ipynb
│   ├── 03_cohort_retention.ipynb
│   └── 04_rfm_segmentation.ipynb
├── tools/
│   ├── rebuild_corrected_outputs.py
│   ├── validate_corrected_project.py
│   └── render_professional_report.py
├── powerbi/                       # original PBIX; refresh from corrected outputs
├── docs/                          # original PDF retained as a legacy snapshot
└── Argos_Ecommerce_Professional_Analysis_Report_ChartOptimized.html
```

## 7. Reproduce the Corrected Project

### Python-first reproducible workflow

```bash
python -m pip install -r requirements.txt
python tools/rebuild_corrected_outputs.py
python tools/validate_corrected_project.py
python tools/render_professional_report.py --project-root .
python tools/validate_corrected_project.py
```

The rebuild command regenerates the analytical CSV outputs directly from `data/raw/`; the renderer then creates the final HTML from the corrected project outputs. The final validation command checks only `data/outputs/` and the generated HTML, so no duplicate portfolio evidence directory is required.

### MySQL workflow

Run in order:

1. `sql/01_setup_and_base_tables.sql`
2. `sql/02_funnel.sql`
3. `sql/03_cohort.sql`
4. `sql/04_rfm.sql`
5. `sql/05_profitability.sql`

The SQL scripts implement the same corrected methodology for users who want the database workflow.

### Notebooks

The notebooks now use project-relative paths rather than a hard-coded local Windows directory. Run them after rebuilding `data/outputs/`.

### Power BI

`powerbi/Argos_BI_Report.pbix` is the original project file. Refresh its sources against the corrected `data/outputs/` folder before using its values. The committed screenshot and original PDF are legacy snapshots and may display pre-correction figures; the corrected HTML report is the authoritative rendered report in this package.

## 8. Validation and Audit Files

- `data/quality/data_quality_audit.csv`
- `data/quality/data_quality_summary.json`
- `data/quality/order_items_unmatched.csv`
- `data/quality/order_item_subtotal_reconciliation.csv`
- `data/quality/validation_summary.json`

These files make the major data-quality decisions reviewable rather than implicit.

## Author

**Utkarsh Pandey**  
Data Analyst

---

### Correction note

This edition was rebuilt so that conclusions are constrained by the project data and calculation logic. Recommendations in the report are framed as hypotheses or experiments where the dataset does not contain causal evidence, campaign cost, or incremental lift.
