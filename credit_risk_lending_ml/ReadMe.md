1. Data Generation & Verification
Running generate_data.py reproducibly outputs the primary datasets:
•	credit_applicants.csv:
o	Total Records: 400 applicant profiles
o	Measured Default Rate: 20.25% (within target range of 15%–25%)
o	Missing Values: Exactly 80 records (20.00%) missing credit_bureau_score (Thin-file applicants)
•	txn_behaviour.csv:
o	Total Records: 265 transactions
o	Seeded Behavioral Anomalies: Exactly 15 transactions prefixed with BTXNA* (~5.66% contamination rate)
2. Thin-File Handling & Preprocessing Pipeline
To prevent target leakage and data snooping, preprocessing steps were strictly sequenced as follows:
1.	Feature Engineering (is_thin_file): Created a binary flag is_thin_file = credit_bureau_score.isna().astype(int) directly from raw data prior to any value imputation or row drops. No records were dropped.
2.	Stratified Train/Test Split: Applied a 75/25 stratified split on the target variable (default) using random_state=42 to guarantee identical class distributions across splits:
o	Train set: 300 applicants (61 defaults, 239 non-defaults)
o	Test set: 100 applicants (20 defaults, 80 non-defaults)
3.	Median Imputation (Strict Train Isolation): Calculated median bureau score on training data only (credit_bureau_score = 645.0) and transformed both training and test sets.
4.	Encoding & Feature Scaling: One-hot encoded categorical variables (employment_type) and fitted StandardScaler strictly on the training set, subsequently transforming both splits.
3. Classifier Evaluation Suite
Both models were trained on identical training splits and evaluated on the 100-record holdout test set:
Performance Metric	Logistic Regression	Decision Tree Classifier
Confusion Matrix [TN, FP, FN, TP]	[69, 11, 13, 7]	[59, 21, 14, 6]
Accuracy	76.00%	65.00%
Precision	38.89%	22.22%
Recall	35.00%	30.00%
F1-Score	36.84%	25.53%
ROC-AUC Score	0.7188	0.5188
4. Risk-Based Pricing Strategy
Using predicted default probabilities from Logistic Regression, test applicants were binned into 4 equal-sized risk tiers (quartiles). The observed default rates demonstrate clear, monotonic risk escalation across tiers:
Risk Tier	Applicant Count	Probability Range (Pdefault)
Observed Defaults	Observed Default Rate	Illustrative APR
Tier 1 (Low Risk)	25	0.49% – 3.58%	2	8.00%	9.5% – 12.0%
Tier 2 (Med-Low Risk)	25	3.63% – 14.61%	3	12.00%	12.5% – 15.5%
Tier 3 (Med-High Risk)	25	15.06% – 33.77%	5	20.00%	16.0% – 20.0%
Tier 4 (High Risk)	25	35.16% – 94.69%	10	40.00%	21.0% – 28.0%
5. Unsupervised Anomaly Detection (IsolationForest)
IsolationForest was fitted on standardized behavioral features (txn_hour, is_new_device, txn_amount_inr) with a contamination rate matched to the ground truth (15 / 265 ≈ 0.0566):
•	Total Seeded Behavioral Anomalies (BTXNA*): 15
•	Anomalies Successfully Flagged: 11
•	Isolation Forest Recall: 73.33%
6. Bias-Awareness & Governance Note
•	Correlated-Proxy Risk: employment_type and is_thin_file often correlate strongly with protected demographic attributes (such as age or socio-economic background). Relying heavily on thin-file flags or employment category without calibration risks systematic adverse impact against young or unbanked populations.
•	Governance Mitigation Step: Implement periodic Disparate Impact (Four-Fifths Rule) audits across protected demographic segments and establish a manual human-in-the-loop (HITL) review policy for automated rejections originating from thin-file inputs.
7. Final Model Comparison & Recommendation
•	Recommended Production Model: Logistic Regression
•	Justification:
o	ROC-AUC Superiority: Logistic Regression achieved an AUC of 0.7188, significantly outperforming the Decision Tree (0.5188), which suffered from severe overfitting on training data.
o	Monotonic Risk Grading: Logistic Regression produced calibrated probabilities that separated risk into monotonic tiers (8% default in Tier 1 vs. 40% in Tier 4), enabling risk-based pricing.
o	Interpretability: Provides transparent, auditable feature coefficients required for fair lending compliance and regulatory adverse action notices.

