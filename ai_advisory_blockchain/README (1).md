# Module Overview: AI Advisory & Blockchain Risk Framework (`/ai_advisory_blockchain`)

This module contains core agent execution loops, valuation calculators, NLP signal extractors, and risk frameworks:

```
├── stock_universe.py          # Static asset universe (beta, analyst returns, standard deviation)
├── investor_profiles.py       # Investor profile definitions (INV01 through INV05)
├── disclosure_snippets.py     # Sample regulatory & financial filing text disclosures
├── advisory_agent.py          # Think/Act/Observe agent loop with CAPM & escalation checks
├── extract_disclosure.py     # Schema-validated disclosure signal and sentiment extractor
├── debate.py                  # 3-Agent Bull/Bear/Synthesizer multi-agent debate engine
├── dcf_calculator.py          # 5-Year Unlevered FCFF DCF model & sensitivity grid
├── blockchain_risk_note.md    # T.A.N.G. framework analysis & institutional crypto strategy
└── README.md                  # Module documentation & recorded run transcripts
```

---

## 1. Execution Environment & Control Mode

* **Selected Mode:** `MOCK_LLM = 1` (Default baseline)
* **Configuration:** All agent loops, NLP extractors, and debate components executed strictly in deterministic mock mode with zero network calls, guaranteeing reproducible evaluation runs.

---

## 2. Advisory Agent Execution Transcripts (`MOCK_LLM=1`)

The advisory agent follows a strict three-stage execution pattern:
1. **Think:** Map investor risk tolerance to prescribed equal-weighted asset allocations ($rac{1}{3}$ weight each).
2. **Act:** Retrieve asset parameters from `STOCK_UNIVERSE` via tool call `get_stock_data(ticker)`.
3. **Observe:** Compute portfolio CAPM return $E[R_p] = R_f + eta_p (E[R_m] - R_f)$ using $R_f = 4.0\%$, $E[R_m] = 10.0\%$, and portfolio standard deviation assuming pairwise asset correlation $ho = 0.30$.
4. **Escalation Gate:** If portfolio volatility exceeds $20.00\%$, flag status as `ESCALATED_TO_HUMAN_ADVISOR`; otherwise, `FINALIZED`.

### Recorded Execution Run

| Investor ID | Risk Tier | Allocations / Tickers | CAPM Return | Volatility | Status |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **INV01** | Conservative | PAYBOND, PAYGOLD, PAYRETAIL | 6.20% | 8.44% | `FINALIZED` |
| **INV02** | Moderate | PAYRETAIL, PAYINFRA, PAYGOLD | 8.30% | 12.57% | `FINALIZED` |
| **INV03** | Aggressive | PAYTECH, PAYFIN, PAYINFRA | 12.00% | 20.58% | `ESCALATED_TO_HUMAN_ADVISOR` |
| **INV04** | Moderate | PAYRETAIL, PAYINFRA, PAYGOLD | 8.30% | 12.57% | `FINALIZED` |
| **INV05** | Aggressive | PAYTECH, PAYFIN, PAYINFRA | 12.00% | 20.58% | `ESCALATED_TO_HUMAN_ADVISOR` |

### Deterministic Narratives
* **INV01:** *"For Conservative investor INV01, we recommend an allocation across PAYBOND, PAYGOLD, PAYRETAIL with an expected portfolio return of 6.20% and volatility of 8.44%."*
* **INV03:** *"For Aggressive investor INV03, we recommend an allocation across PAYTECH, PAYFIN, PAYINFRA with an expected portfolio return of 12.00% and volatility of 20.58%."*

---

## 3. Regulatory Disclosure Signal Extraction (`extract_disclosure.py`)

Extracted signals across 6 test disclosures running in offline mock mode (`MOCK_LLM=1`):

```json
{
  "doc_01": {
    "risk_flags": [],
    "hedging_detected": true,
    "sentiment": "cautious"
  },
  "doc_02": {
    "risk_flags": ["litigation"],
    "hedging_detected": false,
    "sentiment": "neutral"
  },
  "doc_03": {
    "risk_flags": ["customer concentration"],
    "hedging_detected": false,
    "sentiment": "neutral"
  },
  "doc_04": {
    "risk_flags": [],
    "hedging_detected": true,
    "sentiment": "cautious"
  },
  "doc_05": {
    "risk_flags": [],
    "hedging_detected": false,
    "sentiment": "confident"
  },
  "doc_06": {
    "risk_flags": ["regulatory"],
    "hedging_detected": false,
    "sentiment": "neutral"
  }
}
```

---

## 4. 3-Agent Investment Debate Demo (`debate.py`)

Execution transcript for target ticker **PAYTECH** (Analyst Return: 19.0%, Beta: 1.55, Volatility: 34.0%):

```json
{
  "ticker": "PAYTECH",
  "metrics": {
    "analyst_expected_return": "19.0%",
    "beta": "1.55",
    "std_dev": "34.0%"
  },
  "bull_agent": "With an expected return of 19.0% against a beta of 1.55, this offers attractive risk-adjusted upside.",
  "bear_agent": "However, a high volatility of 34.0% presents substantial downside risk and severe stock price instability.",
  "synthesizer": "PAYTECH presents a high-reward opportunity with an expected return of 19.0%. While its beta of 1.55 indicates strong upside potential during market rallies, investors must carefully weigh this against its elevated volatility of 34.0%. A measured position size is recommended."
}
```

---

## 5. DCF Valuation & Sensitivity Analysis (`dcf_calculator.py`)

### Model Parameters
* **Base Unlevered FCFF:**  
  $$	ext{EBIT}(1 - t) + 	ext{D\&A} - 	ext{CapEx} - \Delta	ext{NWC} = 1000(1 - 0.25) + 150 - 200 - 50 = \mathbf{₹650.00	ext{ Mn}}$$
* **PAYRETAIL Cost of Equity ($r_e$):**  
  $$r_e = 4.0\% + 0.85 	imes (10.0\% - 4.0\%) = 9.10\%$$
* **Base WACC:**  
  $$	ext{WACC} = (80\% 	imes 9.10\%) + (20\% 	imes 6.00\%	ext{ after-tax}) = 8.48\%$$
* **5-Year Growth Profile:** $12\%, 10\%, 8\%, 6\%, 4\%$ (Terminal Growth $g = 3.00\%$)
* **Base Enterprise Value (DCF):** **₹15,254.48 Mn**

### $3 	imes 3$ WACC vs. Terminal Growth Sensitivity Grid
*(Enterprise Value in ₹ Mn)*

| WACC \ Terminal $g$ | 2.00% | 3.00% (Base) | 4.00% |
| :--- | :---: | :---: | :---: |
| **7.48% (-1.0%)** | ₹15,791.65 | ₹18,701.81 | ₹23,284.49 |
| **8.48% (Base)** | ₹13,315.68 | **₹15,254.48** | ₹18,058.82 |
| **9.48% (+1.0%)** | ₹11,502.81 | ₹12,872.03 | ₹14,740.95 |

> **Note:** In every grid cell, WACC exceeds terminal growth ($	ext{WACC} > g$), preventing mathematical divergence.

### EV/EBITDA Multiples Cross-Check
* **Base EBITDA:** $	ext{EBIT} + 	ext{D\&A} = 1000 + 150 = \mathbf{₹1,150.00	ext{ Mn}}$
* **12.0x Multiple Valuation:** $1,150 	imes 12.0 = \mathbf{₹13,800.00	ext{ Mn}}$
* **Comparative Note:** The intrinsic DCF value (₹15,254.48 Mn) trades at an $\sim 10.5\%$ premium relative to the static multiples cross-check (₹13,800.00 Mn). This premium is economically justified by the strong near-term organic FCFF compound annual growth rate ($\sim 10\%$ over 5 years) captured in the DCF.
