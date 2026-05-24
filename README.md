# stress-weighted-sparse-saa

Stress-weighted sparse strategic asset allocation with soft-Frobenius covariance estimation, robust growth constraints, and greedy core-satellite portfolio construction.

## Overview

This repository develops a research framework for **stress-aware strategic asset allocation (SAA)**.

The core idea is that standard mean-variance portfolio construction usually relies on an unconditional covariance matrix, even though diversification benefits may deteriorate precisely when they are most needed. This project addresses that problem by estimating a **stress-weighted covariance matrix** using a continuous regime signal based on the Frobenius distance between the current and long-run correlation structures.

The resulting covariance estimator is then used inside a robust portfolio construction framework with:

- a fixed equity core;
- sparse greedy satellite selection;
- robust log-growth constraints;
- walk-forward validation;
- transaction-cost robustness;
- ablation tests against Markowitz-style benchmarks.

The project is intended as a research prototype for **portfolio construction, strategic asset allocation, stress-aware risk estimation, and compound-growth preservation**.

---

## Research Question

Can a strategic asset allocation framework improve drawdown-adjusted and compound-growth-aware performance by:

1. identifying when the dependence structure of asset classes departs from its long-run configuration;
2. estimating a covariance matrix that gives more weight to such stress-like observations;
3. constructing a sparse, interpretable core-satellite allocation under robust growth constraints?

---

## Core Methodology

### 1. Frobenius stress signal

The regime state variable is defined as the distance between the current rolling or EWMA correlation matrix and a long-run reference matrix:

```math
D_t = \frac{\|R_t - \bar{R}\|_F}{\sqrt{N(N-1)}}
```

where:

- `R_t` is the current estimated correlation matrix;
- `R_bar` is the long-run reference correlation matrix;
- `N` is the number of asset classes.

The signal is designed to capture **dependence-structure instability**, not return predictability.

---

### 2. Soft stress weighting

Instead of classifying days as stress/non-stress through a hard percentile threshold, observations are weighted continuously:

```math
\omega_t(\kappa) =
\frac{\exp(\kappa z_t)}
{\sum_s \exp(\kappa z_s)}
```

where `z_t` is the standardized Frobenius stress signal.

The parameter `kappa` controls the intensity of stress conditioning:

- `kappa = 0`: unconditional covariance;
- higher `kappa`: greater emphasis on stress-like observations.

To avoid over-concentration on too few observations, the framework tracks the effective sample size:

```math
N_{eff} = \frac{1}{\sum_t \omega_t^2}
```

---

### 3. Stress-weighted covariance estimation

The stress-weighted covariance estimator is:

```math
\Sigma_{\kappa}
=
\sum_t \omega_t(\kappa)
(r_t - \bar{r}_{\omega})(r_t - \bar{r}_{\omega})^\top
```

The estimator is then stabilized using shrinkage.

This separates two tasks:

- **stress conditioning** via soft regime weights;
- **estimation-error control** via shrinkage.

---

### 4. Robust log-growth constraint

The optimization does not rely on point estimates of expected returns alone.

Instead, expected log-growth is estimated through a bootstrap distribution, and the portfolio must satisfy a lower-bound-style robust growth constraint:

```math
w^\top \mu_g - k \sqrt{w^\top \Sigma_g w} \geq \tau
```

where:

- `mu_g` is the estimated mean log-growth vector;
- `Sigma_g` is the covariance of bootstrapped growth estimates;
- `k` controls conservativeness;
- `tau` is the required robust growth threshold.

---

### 5. Greedy sparse core-satellite construction

The framework starts from a fixed equity core and adds satellite assets sequentially.

At each step, the algorithm selects the asset that most improves the stress-aware portfolio objective while preserving the robust growth constraint.

The greedy layer is not treated as a formal globally optimal algorithm. Its role is to provide:

- sparse allocations;
- interpretability;
- reduced sensitivity to noisy dense optimizer outputs;
- an economically meaningful core-satellite construction path.

---

## Model Variants and Ablation

The repository compares four main model classes:

| Model | Covariance | Construction | Purpose |
|---|---|---|---|
| Markowitz Base-Cov | unconditional/base covariance | continuous optimizer | classical baseline |
| Stress-MV Soft-Frob | soft-Frobenius stress covariance | continuous optimizer | isolates value of stress-weighted covariance |
| Greedy Base-Cov | unconditional/base covariance | greedy sparse allocation | isolates value of greedy sparsification |
| Greedy Soft-Frob | soft-Frobenius stress covariance | greedy sparse allocation | full framework |

This ablation is central to the project. It distinguishes whether performance improvements come from:

- the stress-weighted covariance estimator;
- the sparse greedy allocation layer;
- or their interaction.

---

## Asset Universe

The current implementation uses liquid US-listed ETF proxies for broad asset classes:

```python
TICKERS = [
    "SPY",   # US equity
    "EFA",   # developed ex-US equity
    "EEM",   # emerging markets equity
    "SHY",   # US Treasuries 1-3Y
    "IEF",   # US Treasuries 7-10Y
    "TLT",   # US Treasuries 20+Y
    "TIP",   # US TIPS
    "LQD",   # investment grade credit
    "HYG",   # high yield credit
    "GLD",   # gold
    "DBC",   # broad commodities
    "VNQ",   # REITs
]
```

The universe is deliberately simple and transparent. It is meant to test the framework on broad investable asset classes, not to optimize a production allocation.

---

## Validation Design

The framework includes:

- train / validation / test splits;
- paper-safe `kappa` selection using validation performance and effective-sample-size constraints;
- walk-forward analysis;
- walk-forward protocol sensitivity across rolling and expanding windows;
- subperiod analysis;
- bootstrap inference on performance differentials.

The main evaluation metrics include:

- CAGR / annualized return;
- volatility;
- Sharpe ratio;
- maximum drawdown;
- Calmar ratio;
- Ulcer Index;
- Martin Ratio;
- turnover;
- transaction-cost-adjusted performance.

The Martin Ratio and Ulcer Index are emphasized because the project focuses on **drawdown path quality** and **compound-growth preservation**, not only variance reduction.

---

## Transaction Costs

Transaction costs are modeled using data-driven, asset-specific, time-varying spread estimates.

The current robustness panel includes:

- Corwin-Schultz spread estimator;
- Abdi-Ranaldo-style high-low-close estimator;
- CS/AR median composite;
- CS/AR max composite;
- data-driven spread floors;
- stress-liquidity multipliers linked to the Frobenius regime signal.

The goal is to avoid unrealistic flat transaction costs while testing whether the framework remains robust under alternative liquidity assumptions.

---

## Current Empirical Findings

The latest research runs suggest the following qualitative findings:

1. **Soft-Frobenius stress covariance is a major driver** of improved drawdown-aware allocation in static train/validation/test experiments.
2. **Greedy sparse allocation provides interpretability and robustness**, especially in walk-forward settings where shorter training windows make covariance estimates noisier.
3. The full framework tends to improve **Sharpe, Martin Ratio, Ulcer Index, and maximum drawdown** relative to standard Markowitz-style baselines.
4. The framework does not primarily claim to generate higher raw returns; its value is in improving the **risk-adjusted and compounding-aware profile** of strategic allocations.
5. The method appears especially relevant when the correlation structure departs materially from its long-run configuration.

These results are preliminary and should be interpreted as research findings, not investment advice.

---

## Repository Structure

Suggested structure:

```text
stress-weighted-sparse-saa/
│
├── README.md
├── requirements.txt
├── notebooks/
│   └── stress-weighted-sparse-saa.ipynb
│
├── src/
│   ├── data.py
│   ├── stress_signals.py
│   ├── covariance.py
│   ├── optimization.py
│   ├── backtesting.py
│   ├── transaction_costs.py
│   └── metrics.py
│
├── figures/
│   ├── frontier_comparison.png
│   ├── drawdown_comparison.png
│   ├── walk_forward_results.png
│   └── stress_signal.png
│
└── results/
    ├── ablation_table.csv
    ├── transaction_cost_robustness.csv
    ├── walk_forward_summary.csv
    └── subperiod_analysis.csv
```

At the current stage, the main implementation is notebook-based. Refactoring into reusable Python modules is planned.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/<your-username>/stress-weighted-sparse-saa.git
cd stress-weighted-sparse-saa
```

Create a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
# .venv\Scripts\activate    # Windows
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Suggested packages:

```text
numpy
pandas
scipy
scikit-learn
cvxpy
matplotlib
yfinance
statsmodels
tqdm
```

Depending on the solver used by CVXPY, you may also need:

```text
ecos
scs
clarabel
```

---

## Running the Notebook

Open the main notebook:

```text
notebooks/stress-weighted-sparse-saa.ipynb
```

The notebook performs:

1. data download from Yahoo Finance;
2. return and log-return construction;
3. Frobenius stress signal estimation;
4. soft stress-weighted covariance estimation;
5. robust growth estimation;
6. greedy and continuous optimization;
7. ablation study;
8. transaction-cost robustness;
9. walk-forward sensitivity;
10. inference and diagnostics.

Some sections can be computationally expensive. Walk-forward sensitivity can be switched off in the notebook if needed.

---

## Methodological Notes

This is a research prototype. The framework is intentionally transparent rather than production-optimized.

Important caveats:

- ETF proxies are used as investable asset-class representatives.
- Results depend on the chosen universe, data vendor, liquidity assumptions, and validation protocol.
- The greedy algorithm is used as an interpretable sparsification layer, not as a formally proven globally optimal algorithm.
- The Frobenius signal is a proxy for dependence-structure instability, not a crisis predictor.
- Transaction-cost estimates are based on daily OHLC data and should be interpreted as approximations.

---

## Roadmap

Planned improvements:

- refactor notebook code into reusable Python modules;
- add automated result export to `results/`;
- add plotting utilities for paper-ready figures;
- test additional asset universes;
- extend inference on frontier-level differences;
- prepare a working paper / SSRN version;
- explore a future multi-period compounding-risk extension.

---

## Research Positioning

This project sits at the intersection of:

- strategic asset allocation;
- robust portfolio construction;
- covariance estimation;
- stress testing;
- sparse allocation;
- drawdown-aware evaluation;
- compound-growth preservation.

A natural applied-paper framing is:

> Stress-weighted covariance estimation for sparse strategic asset allocation.

A possible future methodological extension is:

> Multi-period compounding-aware risk measures for stress-aware portfolio construction.

---

## Disclaimer

This repository is for research and educational purposes only.

It does not provide investment advice, trading recommendations, or portfolio management services. Historical backtests are not indicative of future performance. All results are sensitive to data, assumptions, transaction-cost models, and implementation details.

---

## Author

Saverio Lauriola  
Quantitative finance / portfolio construction / market risk analytics  
GitHub: [@svr-L](https://github.com/svr-L)
