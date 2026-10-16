# Domain Glossary & Quantitative Terminology (`GLOSSARY.md`)

This document defines the canonical domain vocabulary for the `qtrade` quantitative multi-day swing research engine. All technical specs, tests, UI copy, and code symbols must strictly adhere to these definitions.

---

## 1. Regulatory & Safe Harbor Terms (SEBI Mandated)

| Canonical Term | Deprecated / Prohibited Synonyms | Definition & Standard |
|---|---|---|
| **Historical Sample Positive Frequency** | Win Rate, Success Rate, Hit Ratio | The proportion of historical instances in a sample where the upper barrier was reached before the lower or vertical barrier: $\frac{N(y=+1)}{N_{total}}$. |
| **Historical Observed MFE** | Target Price, Take Profit, Profit Target | Maximum Favorable Excursion observed during the multi-day holding period prior to barrier termination. |
| **Historical Observed MAE** | Stop Loss, Max Loss, Cutoff | Maximum Adverse Excursion observed during the multi-day holding period prior to barrier termination. |
| **Historical Observed Barrier Likelihood** | Signal, Prediction, Forecast, Trade Call | The model-estimated probability distribution $P(y = k \mid X_t)$ for $k \in \{+1, 0, -1\}$ or binary meta-label $P(y_{meta} = 1 \mid X_t)$. |
| **Historical Sample Median Forward Return ($p_{50}$)** | Expected Profit, Projected Gain | The median return across all historically matched pattern instances over a defined multi-day horizon. |

---

## 2. Market Structure, Bhavcopy & Feature Vocabulary

| Term | Symbol / Code | Mathematical Definition |
|---|---|---|
| **20-Day Range Location** | `range_pos_20` | Relative location within the trailing 20-day high-low envelope: $\frac{Close_t - Low_{20}}{High_{20} - Low_{20}}$. |
| **Delivery Volume Ratio** | `delivery_ratio` | Daily cash delivery volume divided by its 20-day Simple Moving Average: $\frac{\text{Delivery Vol}_t}{SMA_{20}(\text{Delivery Vol})_t}$. |
| **Delivery Percentage** | `delivery_pct` | The percentage of total traded volume marked for delivery on NSE Cash Bhavcopy: $\frac{\text{Delivery Vol}_t}{\text{Total Traded Vol}_t} \times 100$. |
| **Relative Open Interest** | `rel_oi` | Percentage deviation of Open Interest from its 20-period Simple Moving Average: $\frac{OI_t - SMA_{20}(OI)_t}{SMA_{20}(OI)_t}$. |
| **3-Day OI Expansion** | `oi_expansion_3d` | Net percentage change in futures Open Interest over a 3-day window: $\frac{OI_t - OI_{t-3}}{OI_{t-3}}$. |
| **Normalized Daily ATR** | `norm_atr` | 14-day Average True Range expressed as a percentage of closing price: $\frac{ATR_{14, t}}{Close_t} \times 100$. |
| **Institutional OI Regimes** | `oi_regime` | Categorical classification: **Long Buildup** ($P\uparrow, OI\uparrow$), **Short Covering** ($P\uparrow, OI\downarrow$), **Short Buildup** ($P\downarrow, OI\uparrow$), **Long Unwinding** ($P\downarrow, OI\downarrow$). |

---

## 3. Execution Seam & Multi-Day Triple Barrier Method

| Term | Symbol / Code | Definition |
|---|---|---|
| **Signal Seam** | `signal_time` | Calculated strictly at Market Close (15:30 IST) of Day $T$ using finalized Daily Bhavcopy and F&O data. |
| **Execution Seam** | `exec_time` | Simulated execution occurs at Market Open (09:15–09:20 IST) of Day $T+1$ with a mandatory $0.10\%$ half-spread and slippage deduction. |
| **Upper Horizontal Barrier** | `upper_barrier` ($R^+$) | Dynamic target set at Entry $+ (2.0 \times \text{Daily } ATR_{14})$. Touched first triggers label $y = +1$. |
| **Lower Horizontal Barrier** | `lower_barrier` ($R^-$) | Dynamic stop set at Entry $- (1.0 \times \text{Daily } ATR_{14})$. Touched first triggers label $y = -1$. |
| **Vertical Time Barrier** | `vertical_barrier` ($T$) | Expiration time limit set at 3 trading days (Close of Day $T+3$). Expiration prior to horizontal touch triggers label $y = 0$. |
| **Gap-Over Stop Execution** | `gap_stop_fill` | When Day $T+2$ opens below the lower barrier, the fill price is the actual Open price, recording the true real-world gap loss. |

---

## 4. Modeling & Research Methodology

| Term | Definition |
|---|---|
| **Zero-ML Baseline** | A transparent, non-machine-learning rule (20-day Breakout + Delivery Ratio $>1.5$ + OI Expansion $>5\%$) serving as the benchmark that all subsequent experiments must beat. |
| **Meta-Labeling** | A two-stage framework where a primary rule proposes trade candidates, and a secondary ML model (LightGBM) predicts the probability of barrier success ($y_{meta} \in \{0, 1\}$) to filter false breakouts. |
| **Purged Walk-Forward Cross-Validation** | Rolling time-series validation (e.g., 3-year train / 1-year test) where overlapping multi-day trade horizons are purged at the train/test seam. |
| **Sample Uniqueness Weighting ($u_i$)** | Weighting each training sample inversely to the number of concurrent active trades to eliminate serial redundancy. |
| **Deflated Sharpe Ratio (DSR)** | A statistical metric that adjusts the observed out-of-sample Sharpe ratio for selection bias and the total number of trial experiments conducted. |
