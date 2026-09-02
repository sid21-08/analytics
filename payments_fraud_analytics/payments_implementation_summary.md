# Project Steps & Implementation Summary

**Module Target:** Payment Reconciliation, Fraud Analytics & Merchant Performance  
**Execution Seed:** `42` (Deterministic Synthetic Data Generation)

---

## 1. Data Generation (`generate_data.py`)

Executing `generate_data.py` with `seed=42` reproducibly generates the primary synthetic payment datasets:

* **`merchants.csv`:** 40 merchant profiles
* **`users.csv`:** 365 user accounts (350 established accounts + 15 burner accounts)
* **`ledger.csv`:** 547 ledger entries (500 baseline + 15 burner chargebacks + 32 velocity attack transactions)
* **`gateway_export.csv`:** Discrepant export file containing injected anomalies:
  * **Missing Records:** $\sim 5\%$ missing in gateway
  * **Amount Mismatches:** $\sim 3\%$ amount discrepancy
  * **Status Mismatches:** $\sim 2\%$ status discrepancy
  * **Ledger-Missing Records:** $\sim 2\%$ extra gateway-only records

---

## 2. Part A — Excel Merchant Workbook (`merchant_workbook.xlsx`)

* **`VLOOKUP` Enrichment:** Applied fixed absolute references (`$A$2:$D$41`) to map `merchant_name`, `category`, and `region` into the primary transactions view, wrapped in `IFERROR(..., "Merchant not found")`.
* **`HLOOKUP` Fee Tier Lookup:** Built a horizontal Merchant Discount Rate (MDR) reference table mapping payment methods to fee rates and populated transaction fee structures:
  * **UPI:** 0.00%
  * **Wallet:** 1.10%
  * **Card:** 1.90%
  * **Netbanking:** 1.50%
* **Nested `IF`/`AND` Classification:** Added a dynamic rule labeling transactions as `"High-Value Merchant Day"` if `merchant_daily_pivot_total > 5000` **AND** `region != "East"`.
* **Pivot Table Analysis:** Summarized aggregate `amount_inr` and transaction counts grouped by `merchant_id` and `status`. Included a distinct count comparison (`count-vs-count-unique`) analyzing transacted days against total transaction volume across merchants.

---

## 3. Part B — SQL Database & Fraud Detection (`paytm_payments.db`)

Built a normalized SQLite schema establishing `merchants`, `users`, and `transactions` tables with primary/foreign key constraints. Executed 6 analytical queries leveraging core SQL clauses (`SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`, `DISTINCT`, `INNER JOIN`, `LEFT JOIN`):

1. **Chargeback Impact Quantification:** Calculated aggregate chargeback transaction volume, distinct users impacted, and cumulative lost GMV.
2. **Burner Account Detection:** Identified transactions where `status = 'chargeback'` and $0 \le (	ext{transaction\_time} - 	ext{signup\_date}) < 30 	ext{ days}$, successfully capturing all **15 seeded burner account rows**.
3. **Velocity Attack Detection:** Grouped transactions by `user_id` and 10-minute time buckets (`strftime('%s', transaction_time) / 600`) with `HAVING count >= 3`, successfully isolating all **8 seeded velocity clusters**.
4. **Regional GMV Breakdown (`INNER JOIN`):** Aggregated captured GMV across merchant geographic regions.
5. **Uncaptured / Failed Analysis (`LEFT JOIN`):** Isolated user accounts experiencing transaction failures across payment channels.
6. **Payment Method Success Rates (`GROUP BY` / `HAVING`):** Filtered channels handling high transaction volumes to calculate conversion rates.

---

## 4. Part C — Python Payment Reconciliation (`reconcile.py`)

Engineered a modular function `reconcile_payments(ledger_df, gateway_df)` utilizing set symmetric differences (`ledger_ids ^ gateway_ids`) and pairwise merges (`pd.merge`). Isolated 4 distinct discrepancy DataFrames matching target injected rates:

* **Missing in Gateway:** 27 rows ($\sim 5\%$)
* **Missing in Ledger:** 10 rows ($\sim 2\%$)
* **Amount Mismatches:** 16 rows ($\sim 3\%$)
* **Status Mismatches:** 10 rows ($\sim 2\%$)

---

## 5. Part D — Analytics Dashboard & Scorecards (`paytm_dashboard.png`)

Rendered a four-layer dashboard saved as a high-resolution image (`paytm_dashboard.png`):

### A. Headline Layer (Core Scorecards)

| Metric | Recorded Value |
| :--- | :---: |
| **Total GMV** | **₹724,153.00** |
| **Overall Success Rate** | **89.03%** |
| **Exact Match Rate** | **89.76%** |
| **Headline Chargeback Ratio** | **4.57%** |

### B. Analytical Layers
* **Trends Layer:** Dual-axis daily time-series chart tracking GMV against chargeback occurrences over a 30-day window.
* **Breakdown Layer:** Stacked bar chart breakdown displaying GMV distribution across payment methods and merchant categories.
* **Details Layer:** Image-rendered summary table displaying the **Top 10 Merchants** by transaction volume with conditional high-risk flags applied when `chargeback_ratio > 1.0%`.
