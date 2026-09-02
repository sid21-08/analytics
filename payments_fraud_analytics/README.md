Project Steps & Implementation Summary
1. Data Generation (generate_data.py)

Executed script with seed 42 to produce synthetic payment datasets:

merchants.csv (40 rows)

users.csv (365 rows: 350 established + 15 burner accounts)

ledger.csv (547 rows: 500 baseline + 15 burner chargebacks + 32 velocity attack transactions)

gateway_export.csv (Discrepant export with injected ~5% missing, ~3% amount mismatch, ~2% status mismatch, and ~2% ledger-missing records)

2. Part A — Excel Merchant Workbook (merchant_workbook.xlsx)

VLOOKUP Enrichment: Added fixed absolute references ($A$2:$D$41) to map merchant_name, category, and region into transactions-view, using IFERROR(..., "Merchant not found").

HLOOKUP Fee Tier Lookup: Built a horizontal MDR reference table for payment methods (UPI: 0.00%, Wallet: 1.10%, Card: 1.90%, Netbanking: 1.50%) and pulled fee rates per transaction.

Nested IF/AND Classification: Added rule column labeling transactions as "High-Value Merchant Day" if merchant daily pivot total > INR 5,000 AND region is NOT "East".

Pivot Table Analysis: Summarized total amount_inr and transaction count grouped by merchant_id and status, including a distinct count comparison (count-vs-count-unique) of transacted days vs. total transaction count across merchants.

3. Part B — SQL Database & Fraud Detection (paytm_payments.db)

Built a normalized SQLite schema defining merchants, users, and transactions tables with primary/foreign key constraints.

Executed 6 core analytical queries covering all SQL clauses (SELECT, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT, DISTINCT, INNER JOIN, LEFT JOIN):

Chargeback Impact Quantification: Calculated total chargeback transaction volume, distinct users impacted, and cumulative lost GMV.

Burner Account Detection: Identified transactions where status = 'chargeback' and 0 <= (transaction_time - signup_date).days < 30, successfully capturing all 15 seeded burner account rows.

Velocity Attack Detection: Grouped transactions by user_id and 10-minute time buckets (strftime('%s', transaction_time) / 600) with HAVING count >= 3, successfully isolating all 8 seeded velocity clusters.

Regional GMV Breakdown (INNER JOIN): Aggregated captured GMV by merchant region.

Uncaptured / Failed Analysis (LEFT JOIN): Identified user accounts experiencing transaction failures across payment channels.

Payment Method Success Rates (GROUP BY / HAVING): Filtered channels handling high transaction volumes.

4. Part C — Python Payment Reconciliation (reconcile.py)

Developed modular reconcile_payments(ledger_df, gateway_df) utilizing set operations (ledger_ids ^ gateway_ids) and pairwise merges (pd.merge).

Returned 4 discrepancy DataFrames matching injected rates:

Missing in Gateway: 27 rows (~5%)

Missing in Ledger: 10 rows (~2%)

Amount Mismatches: 16 rows (~3%)

Status Mismatches: 10 rows (~2%)

5. Part D — Analytics Dashboard & Scorecards (paytm_dashboard.png)

Rendered a four-layer saved dashboard image:

Headline Layer: Printed core scorecards using exact metrics:

Total GMV: ₹724,153.00

Overall Success Rate: 89.03%

Exact Match Rate: 89.76%

Headline Chargeback Ratio: 4.57%

Trends Layer: Dual-axis daily time-series chart mapping GMV vs. Chargeback occurrences over 30 days.

Breakdown Layer: Stacked bar chart breakdown of GMV by payment method and merchant category.

Details Layer: Image-rendered summary table of the Top 10 Merchants by transaction volume, applying conditional high-risk flags (chargeback_ratio > 1.0%).
