# Business Requirements Document (BRD)
## Quantitative Multi-Day Swing & Barrier Research Engine (`qtrade`)

- **Document Version:** 2.0.0
- **Status:** Approved Baseline
- **Execution Mode:** Manual Trading Support & Research Engine (Evening Analysis $\rightarrow$ Next-Day Execution)
- **Primary Holding Period:** 1 to 3 Trading Days (Multi-Day Swing)
- **Timeframe Resolution:** Daily & 75-Minute Interval Bars (5 clean bars per NSE trading day)
- **Capital Baseline:** ₹5,00,000 (₹5 Lakhs INR)
- **Asset Universe:** Top 100 Liquid NSE Cash Equities & NIFTY/BANKNIFTY Futures
- **Historical Data Range:** 5 Years (2020 – 2025)

---

## 1. Executive Summary & Problem Definition

### 1.1 The High-Frequency Retail Pitfall
Most algorithmic trading retail projects attempt sub-15-minute or 5-minute scalping. In Indian markets, this fails due to:
1. **Frictional Obliteration:** On a 5-minute chart, an expected move is 25–40 points on NIFTY or 0.5% on a stock. After accounting for STT, exchange turnover charges, GST, stamp duty, brokerage, and bid-ask spread crossing, transaction costs consume **20% to 30% of gross profits**.
2. **Execution Latency Mismatch:** Human manual traders cannot compete with colocated HFT infrastructure on sub-minute horizons.
3. **Overnight Gap Amnesia:** Naive intraday systems that accidentally hold overnight get wiped out when indices or stocks gap 1%–3% against their fixed stops.

### 1.2 The Multi-Day Quantitative Swing Solution
`qtrade` pivots to a **1 to 3-day swing horizon** designed for calm, systematic manual execution:
- **Frictional Efficiency:** On a 1–3 day swing, the price target is $1.5\text{x}$ to $2.5\text{x}$ Daily $ATR$ ($1.5\% - 3.5\%$ on liquid stocks). Total transaction friction drops to **$< 3\%$ of gross gains**, immediately unlocking positive mathematical expectancy.
- **Workflow Fit:** The engine runs after market close (15:30 IST) or at 75-minute intervals, scanning the Top 100 liquid NSE stocks and F&O data. The trader reviews candidate setups in the evening and places orders at the market open (09:15–09:20 IST).
- **Capital Preservation on ₹5L:** Rather than risking margin calls on single-stock futures (where 1 lot requires ₹1.5L–₹2.5L), the system trades **Cash Equities** split into 5 tranches of ₹1,00,000 each (20% allocation per stock, 1%–2% risk per trade), or strictly 1 lot of NIFTY index futures.
- **Scientific 2-Phase Methodology:** Establish a transparent, zero-ML mathematical baseline first. Then, run controlled machine learning experiments (Meta-Labeling, Purged Walk-Forward CV) to prove whether the baseline can be statistically beaten over a 5-year historical horizon.

---

## 2. Regulatory Guardrails & SEBI Safe Harbor (§8.2D)

Under SEBI guidelines for research analysts and algorithmic tool providers, the system functions strictly as a **Historical Pattern & Statistical Distribution Search Engine**.

### 2.1 Statutory Positioning
- **No Real-Time Signals or Advisory:** The engine does not provide buy/sell calls, target predictions, or real-time trade signals.
- **Descriptive Historical Distributions:** It calculates historical sample frequencies (e.g., *"When this price structure and OI buildup occurred over the past 5 years across NIFTY 100 stocks, 64% of occurrences reached $+2.0 \times ATR$ before $-1.0 \times ATR$ over a 3-day horizon"*).

### 2.2 Mandatory Safe Harbor Terminology Mapping
All code, UI copy, and specifications must strictly enforce SEBI safe harbor vocabulary:

| Prohibited / Advisory Term | Mandatory SEBI Safe Harbor Term | Mathematical Definition |
|---|---|---|
| "Win Rate" | **Historical Sample Positive Frequency** | $\frac{N(y = +1)}{N_{total}}$ |
| "Target Price / Take Profit" | **Upper Barrier ($R^+$ / Historical MFE Threshold)** | $P_{entry} + (2.0 \times \text{Daily } ATR_{14})$ |
| "Stop Loss" | **Lower Barrier ($R^-$ / Historical MAE Threshold)** | $P_{entry} - (1.0 \times \text{Daily } ATR_{14})$ |
| "Time Exit" | **Vertical Horizon Barrier ($T$)** | Expiration after 3 trading days |
| "Signal / Prediction" | **Historical Pattern Match & Observed Likelihood** | Historical frequency of barrier outcomes |
| "Expected Profit" | **Historical Sample Median Forward Return ($p_{50}$)** | Median return of matched historical occurrences |

---

## 3. Data Architecture & 5-Year Historical Foundation

### 3.1 Data Sources & Ingestion Stack
The system relies on verifiable, accessible primary data covering 2020–2025:

1. **NSE Daily Bhavcopy (Equities & F&O - Free Official Archive):**
   - **Cash Market Bhavcopy:** Daily Open, High, Low, Close, Volume.
   - **Delivery Position Data:** Delivery Volume and Delivery % per security. (Institutional accumulation footprint).
   - **F&O Bhavcopy:** Stock and Index Futures Open Interest (OI), $\Delta OI$, and Option Chain volume/OI distribution.
2. **Multi-Timeframe Candles (Dhan API / Broker Historical APIs):**
   - Daily and 75-minute OHLCV bars.
   - *75-minute Partition:* Five 75-minute candles cleanly partition the NSE trading day (09:15 to 15:30 = 375 minutes: 09:15–10:30, 10:30–11:45, 11:45–13:00, 13:00–14:15, 14:15–15:30) with zero fractional orphan bars.

### 3.2 Universe Definition & Anti-Survivorship Bias
- **Universe:** Top 100 liquid NSE stocks by trailing 6-month average daily turnover (ADT).
- **Dynamic Rebalancing:** Re-evaluated semi-annually. A stock is only eligible for trades in year $Y$ if it met the liquidity threshold at that time, preventing survivorship bias.
- **Index Trading:** NIFTY 50 and BANKNIFTY continuous futures / ETF equivalents.

---

## 4. Quantitative Logic & Market Structure Model

```
                    Price UP (Higher High / Close > EMA20)
                                      ▲
                                      │
           SHORT COVERING             │            LONG BUILDUP
     (Price ↑, OI ↓, Delivery Low)    │    (Price ↑, OI ↑, Delivery High)
     *Fragile rally / Fades quickly   │    *High-Expectancy Multi-Day Long
                                      │
  ────────────────────────────────────┼────────────────────────────────────► OI UP
                                      │
            LONG UNWINDING            │           SHORT BUILDUP
     (Price ↓, OI ↓, Delivery Low)    │    (Price ↓, OI ↑, Delivery High)
     *Healthy pullback or exit        │    *High-Expectancy Short / Hedge
                                      │
                                      ▼
                    Price DOWN (Lower Low / Close < EMA20)
```

### 4.1 Structural Feature Set (Kailash Nadh Simplicity)
No black-box neural networks. Features are transparent, stationary, and economically grounded:

1. **Price Structure & Momentum:**
   - **20-Day Range Location:** $\frac{Close_t - Low_{20}}{High_{20} - Low_{20}}$ (identifying multi-day consolidation breakouts).
   - **Trend Alignment:** $Close_t > EMA_{20, t} > EMA_{50, t}$.
   - **Daily Volatility:** $ATR_{14}$ normalized by price ($\frac{ATR_{14}}{Close}$).
2. **Institutional Open Interest (OI) & Delivery Dynamics:**
   - **Futures OI Expansion:** Percentage change in OI over the last 3 days: $\frac{OI_t - OI_{t-3}}{OI_{t-3}}$.
   - **Cash Delivery Ratio:** $\frac{\text{Delivery Volume}_t}{SMA_{20}(\text{Delivery Volume})_t} > 1.5$ (separating speculative intraday churn from institutional accumulation).
   - **Regime Tag:** Long Buildup vs. Short Covering vs. Short Buildup vs. Long Unwinding.

---

## 5. The Reality-Proof Backtest Engine (Zero Lookahead, Zero Illusions)

To ensure backtested results accurately reflect live manual execution, the simulation strictly enforces real-world execution mechanics.

### 5.1 The Anti-Lookahead Execution Seam
- **Signal Calculation:** Evaluated strictly at **Day $T$ Market Close (15:30 IST)** using finalized Daily and F&O Bhavcopy data.
- **Order Placement:** Simulated entry occurs at **Day $T+1$ Market Open (09:15–09:20 IST)**.
- **Slippage & Spread Penalty:** Every fill applies a mandatory **$0.10\%$ half-spread and slippage deduction** on both entry and exit.
- **Gap-Up Entry:** If Day $T+1$ opens with a gap up, entry occurs at the higher gap Open price, exactly as experienced in real life.

### 5.2 Multi-Day Triple Barrier Method
For a trade entered on Day $T+1$ at Open price $P_0$:
1. **Upper Horizontal Barrier ($R^+$):**
   $$Barrier_{upper} = P_0 + 2.0 \times \text{Daily } ATR_{14}$$
2. **Lower Horizontal Barrier ($R^-$):**
   $$Barrier_{lower} = P_0 - 1.0 \times \text{Daily } ATR_{14}$$
3. **Vertical Time Barrier ($T$):**
   $$Barrier_{time} = \text{Close of Day } T+3 \text{ (holding period: 2 to 3 days)}$$

#### Real-World Gap-Over Stop Execution Rule
- If Day $T+2$ opens with an adverse gap below the Lower Barrier ($Open < Barrier_{lower}$):
  - The trade is exited at the **Open price** (absorbing the full gap loss).
  - The simulation **never** assumes a fill at the theoretical barrier price.

```
Price
  ▲
  │                               ┌── Upper Barrier (+2.0 x ATR) -> y = +1
  │              /\      *touch*  │
  │      /\     /  \    /─────────┘
  │     /  \   /    \  /
Entry*─/────\─/──────\/───────────
  │                               ┌── Lower Barrier (-1.0 x ATR) -> y = -1
  │───────────────────────────────┘
  │
  └─────┴──────────┴──────────┴──────► Time
     Day T+1    Day T+2    Day T+3 (Vertical Horizon -> y = 0)
```

---

## 6. The Scientific 2-Phase Research Framework (5 Years: 2020–2025)

```
                    [ 5-YEAR HISTORICAL DATA: 2020 - 2025 ]
                                      │
                                      ▼
             ┌──────────────────────────────────────────────────┐
             │       PHASE 1: THE ZERO-ML BENCHMARK BASELINE    │
             │  - Transparent Mathematical Rule                 │
             │  - 20-Day Breakout + Delivery Vol > 1.5x         │
             │    + Futures OI Long Buildup > 5%                │
             │  - Next-Day Open Execution with Real Friction    │
             │  - Fixed Triple Barrier (2x ATR / 1x ATR / 3 Days│
             │  - Metrics: Net Expectancy, Drawdown, Sharpe     │
             └──────────────────────────────────────────────────┘
                                      │
                                      ▼
             ┌──────────────────────────────────────────────────┐
             │         PHASE 2: THE EXPERIMENTATION HARNESS     │
             │  - Experiment 1: Meta-Labeling with LightGBM     │
             │    (Predict probability of barrier success)      │
             │  - Experiment 2: Relative Sector Strength filter │
             │  - Experiment 3: Dynamic IV / Market Regime gate │
             └──────────────────────────────────────────────────┘
                                      │
                                      ▼
             ┌──────────────────────────────────────────────────┐
             │       PHASE 3: PURGED WALK-FORWARD EVALUATION    │
             │  - 3-Year Rolling Train, 1-Year OOS Test         │
             │  - Sample Uniqueness Weighting (u_i)             │
             │  - Deflated Sharpe Ratio (DSR) Multi-Trial Check │
             │  - Rule: Only adopt if it beats Baseline OOS!    │
             └──────────────────────────────────────────────────┘
```

### Phase 1: The Transparent Zero-ML Baseline (Milestone 1)
Before writing any machine learning code, establish a pure mathematical benchmark across the Top 100 NSE stocks from 2020 to 2025. All benchmark metrics, calibration values, and scorecards are formally specified in [BASELINE.md](./BASELINE.md):
- **Baseline Entry Condition:**
  1. $Close_T > \max(High_{T-20} \dots High_{T-1})$ (20-day price breakout).
  2. $\text{Delivery Volume}_T > 1.5 \times SMA_{20}(\text{Delivery Volume})_T$.
  3. Stock Futures $\Delta OI_T > +5\%$ (Long Buildup).
- **Baseline Execution:** Enter at Day $T+1$ Open with $0.10\%$ slippage. Exit via 2x ATR / 1x ATR / 3-Day Triple Barrier.
- **Baseline Reporting:** Calmar Ratio ($\ge 1.75$), Maximum Portfolio Drawdown ($\le 9.5\%$), Net Mathematical Expectancy ($E_{net} \ge +0.22 \text{ ATR}$), Profit Factor ($\ge 1.45$), and Deflated Sharpe Ratio ($> 0.95$).

### Phase 2: Controlled Experiments to Beat the Baseline (Milestones 2+)
Every new idea is tested as a scientific experiment against the baseline. **Not all metrics need to improve—the primary goal is an optimal balance of returns and drawdowns.**

#### The Primary Win Criteria (From BASELINE.md)
An experiment is declared a **WIN** if:
$$\text{WIN} \iff \Delta \text{Calmar Ratio} \ge +20\% \quad \text{AND} \quad \text{Max Drawdown} \le \text{Baseline Max Drawdown}$$
- *Drawdown reduction is favored over reckless returns.* An experiment that maintains $\approx 20\%$ CAGR while halving Drawdown from $9.5\% \rightarrow 5.5\%$ (boosting Calmar from $2.1 \rightarrow 3.6$) is a decisive win.
- Experiments must maintain Net Expectancy $E_{net} \ge +0.20 \text{ ATR}$ and Deflated Sharpe Ratio $> 0.95$.

#### Planned Experiments:
1. **Experiment 1 (López de Prado Meta-Labeling):**
   - The baseline rule proposes candidates.
   - A shallow LightGBM classifier (`max_depth: 3-5`) evaluates microstructure context (Volume Delta, VWAP distance, relative volatility) to predict:
     $$P(\text{Trade hits Upper Barrier} \mid X_T)$$
   - Trades are filtered: take only proposals where $P \ge 0.60$.
   - **Hypothesis:** Improves Net Expectancy and reduces drawdown by filtering out low-conviction false breakouts.
2. **Experiment 2 (Sector Momentum Filter):**
   - Only take stock breakout setups if the stock's corresponding NSE Sector Index is outperforming NIFTY 50 over the trailing 5 days.
3. **Experiment 3 (Market Regime Filter):**
   - Filter trades based on whether NIFTY 50 is above or below its 50-day EMA.

### Phase 3: Validation Rigor & Anti-Overfitting
- **Purged Walk-Forward Cross-Validation:** Rolling 3-year train, 1-year test splits. Overlapping forward trade horizons are purged at fold seams.
- **Sample Uniqueness Weighting ($u_i$):** Training samples are weighted inversely to concurrent overlapping trades.
- **Deflated Sharpe Ratio (DSR):** Backtests must compute DSR to penalize for the number of experimental variations tested. An experiment is declared a failure if its OOS performance does not statistically exceed the Phase 1 Baseline as defined in `BASELINE.md`.

---

## 7. Capital Allocation & Risk Management (₹5,00,000 Baseline)

| Parameter | Specification | Quantitative Rationale |
|---|---|---|
| **Total Account Capital** | ₹5,00,000 (₹5 Lakhs) | Baseline starting capital. |
| **Position Sizing (Cash Stocks)** | **₹1,00,000 per position (5 equal tranches)** | Maximum 5 concurrent positions (20% capital each). |
| **Max Risk per Trade** | **₹1,000 – ₹2,000 (0.2% – 0.4% of total account)** | Stop loss at $1.0 \times ATR$ ($\approx 1\%–2\%$ of position). |
| **Single-Stock Futures Rule** | **Prohibited on ₹5L capital** | Margin concentration and overnight gap risk violates Kelly criterion. |
| **Index Futures Rule** | **Max 1 lot NIFTY** | Margin $\approx ₹1.3\text{L}$; only permitted when cash equity exposure is $< 50\%$. |
| **Portfolio Stop Cutoff** | **Pause trading if account drawdown exceeds 6% (₹30,000)** | Circuit breaker to review model regime drift. |

---

## 8. Technical Stack & Implementation Roadmap

### 8.1 Technology Stack (Kailash Nadh Frugality)
- **Languages:** Python 3.11+, Standard Library.
- **Data Engine:** Polars / Pandas / NumPy / DuckDB for high-throughput vectorized operations.
- **Storage:** Apache Parquet for historical Bhavcopy candles and OI; SQLite for experiment registry and fold metrics.
- **Machine Learning:** LightGBM (`lightgbm.LGBMClassifier`), scikit-learn (purged cross-validation splitters).
- **System Seam:** Decoupled Static API JSON contracts (`data/static/api/v1/`) for inspection dashboards.

### 8.2 Phased Roadmap & SDAD Gate Mapping
- **Milestone 1: Bhavcopy Ingestion & 5-Year Data Pipeline**
  - Gate 1: `specs/requirements/0001-bhavcopy-pipeline.md`
  - Ingests 5 years of daily Bhavcopy, Delivery data, and F&O OI for Top 100 NSE stocks into Parquet.
- **Milestone 2: The Reality-Proof Backtest Engine & Zero-ML Baseline**
  - Gate 1: `specs/requirements/0002-zero-ml-baseline.md`
  - Implements Day $T$ Close $\rightarrow$ Day $T+1$ Open execution, Triple Barrier labeling, and 5-year baseline benchmark report.
- **Milestone 3: Purged Walk-Forward Cross-Validation Harness**
  - Gate 1: `specs/requirements/0003-purged-walk-forward-harness.md`
  - Implements purging, embargoing, and sample uniqueness weighting.
- **Milestone 4: Meta-Labeling Machine Learning Experiment**
  - Gate 1: `specs/requirements/0004-metalabeling-experiment.md`
  - Trains shallow LightGBM models to filter baseline trade proposals, measuring DSR against the baseline.
