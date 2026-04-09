# Lead-Lag Cross-Correlation Study: SPY ↔ QQQ
 
**Intraday Microstructure Analysis & Signal Extraction**
 
---
 
## Abstract
 
This repository implements a lead-lag cross-correlation framework between SPY (SPDR S&P 500 ETF) and QQQ (Invesco Nasdaq-100 ETF) at the 1-minute frequency. The study estimates the directional information flow between the two instruments using Pearson cross-correlation across a symmetric lag window, constructs a threshold-based mean-reversion signal from the identified leader, and evaluates performance via a vectorized backtest with transaction cost adjustment. Results are benchmarked against a passive Buy & Hold allocation.
 
## Data
 
| Parameter       | Value                          |
|-----------------|--------------------------------|
| Instruments     | SPY, QQQ (1-min Close)         |
| Source          | Yahoo Finance (`yfinance`)     |
| Period          | 7 calendar days                |
| Observations    | 2,730                          |
| Frequency       | 1-minute bars (RTH)            |
 
## Methodology
 
### 1. Log Returns & Standardization
 
Returns are computed as log-differences to ensure stationarity and additive aggregation:
 
$$r_t = \ln(P_t) - \ln(P_{t-1})$$
 
Z-scores are derived using full-sample mean and standard deviation for signal generation.
 
### 2. Cross-Correlation Function (CCF)
 
Pearson correlation $\rho_k$ is computed for each integer lag $k \in [-30, +30]$:
 
$$\rho_k = \text{Corr}(r^{\text{SPY}}_t, \; r^{\text{QQQ}}_{t+k})$$
 
The optimal lag is defined as $k^* = \arg\max_k |\rho_k|$. Asymmetry between $\rho_{-1}$ and $\rho_{+1}$ is tested to confirm directional information flow.
 
### 3. Signal Construction
 
A position is entered in the lagging instrument when the leader's z-score exceeds a threshold $\tau$:
 
$$\text{signal}_t = \text{sgn}(z^{\text{leader}}_{t - |k^*|}) \cdot \bold{1}\left(|z^{\text{leader}}_{t - |k^*|}| > \tau\right)$$

### 4. Backtest
 
- **Vectorized execution** using `pandas.shift()` to align signals with forward returns.
- **Transaction costs**: 1 basis point (0.0001) deducted per trade.
- **Benchmark**: Buy & Hold SPY over the identical period.
 
## Key Results
 
```
Optimal Lag:    0 min  |  ρ = 0.9850
ρ(lag=-1):      0.0249
ρ(lag=+1):      0.0055
Δ(asymmetry):   0.0194
```
 
### Parameter Sweep (Train/Test Split: 60/40)
 
```
Best Threshold (Train):  2.00σ  |  Train Sharpe: 2.15
Out-of-Sample Sharpe:   -11.65
```
 
### Performance Summary
 
```
Metric                        Lead-Lag      B&H SPY
---------------------------------------------------
Total Net Return               0.0696%      5.5924%
Annualized Sharpe                 0.55         8.48
Max Drawdown                  -0.6634%     -1.9977%
Win Rate                        36.11%       49.43%
Total Trades                        35          N/A
```
 
## Limitations
 
- **Sample size**: 2,730 observations (~7 trading days) is insufficient for robust parameter estimation. Minimum recommended: 3–6 months of 1-min data.
- **Optimal lag at zero**: The contemporaneous correlation of 0.985 dominates; off-zero correlations are marginal (~0.02), leaving minimal exploitable signal.
- **Overfitting risk**: The train Sharpe of 2.15 collapsed to -11.65 out-of-sample, confirming the 2.0σ threshold was fit to noise.
- **Static parameters**: Fixed lag and threshold cannot adapt to intraday regime shifts.
- **Cost model**: The 1bp assumption understates real execution costs (spread, slippage, market impact) at this frequency.
 
## Potential Enhancements
 
| Approach                  | Rationale                                                        |
|---------------------------|------------------------------------------------------------------|
| **Kalman Filter**         | Dynamic estimation of rolling lead-lag coefficient               |
| **Hidden Markov Model**   | Regime-aware threshold adaptation (trending vs. mean-reverting)  |
| **Walk-Forward Optimization** | Rolling train/test windows to evaluate parameter stability   |
| **Basket Extension**      | Test SPY against XLK, SMH, sector ETFs with lower co-integration |
| **Tick-Level Data**       | Higher resolution may reveal sub-minute microstructure effects   |
| **Spread Model**          | Pair-trade the spread directly rather than directional signals   |
 
## Project Structure
 
```
├── README.md
├── notebooks/
│   └── lead_lag_study.ipynb    # Full analysis (Phases 1-5)
├── outputs/
│   ├── ccf_plot.png            # Cross-Correlation Function
│   └── cumulative_returns.png  # Strategy vs B&H equity curves
```
 
## Dependencies
 
```
python >= 3.9
numpy
pandas
yfinance
matplotlib
seaborn
```
 
## Usage
 
```bash
pip install numpy pandas yfinance matplotlib seaborn
jupyter notebook notebooks/lead_lag_study.ipynb
```
 
## Disclaimer
 
This research is for educational and analytical purposes only. It does not constitute investment advice. Past performance on historical or simulated data does not guarantee future results. The authors assume no liability for financial losses incurred from the use of this code or methodology.
 
---
 
*Built as a quantitative microstructure research exercise.*
