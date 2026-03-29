# Derivatives Pricing Engine

A modular options pricing library built in Python, implementing seven pricing models from analytical Black-Scholes to stochastic volatility and jump diffusion. Models are calibrated to live SPY option chain data and validated against market implied volatility surfaces.

**[View Results Gallery](https://kshitijbhandari.github.io/Options-Pricing-and-Volatility-Surface-Modeling-in-Python/)**

---

## Table of Contents

- [Models Implemented](#models-implemented)
- [Key Results](#key-results)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Quickstart](#quickstart)
- [Phase Overview](#phase-overview)

---

## Models Implemented

| Model | Type | Pricing Method | Key Feature |
|-------|------|---------------|-------------|
| Black-Scholes | Analytical | Closed form | Benchmark: flat IV smile |
| Binomial Tree (CRR) | Lattice | Backward induction | European + American exercise |
| Monte Carlo (GBM) | Simulation | Path averaging | Exotic payoffs, variance reduction |
| Finite Difference | PDE | Tridiagonal solve | Explicit, Implicit, Crank-Nicolson |
| Heston | Stochastic Vol | Characteristic function (Lewis 2001) | Volatility smile via correlated vol process |
| Merton Jump Diffusion | Jump process | Series expansion | Fat tails, steep short-term skew |
| SABR | Stochastic Vol | Hagan et al. approximation | Per-slice calibration, near-perfect ATM fit |

**Exotic options:** Asian calls/puts (arithmetic average payoff), barrier options (down-and-out, down-and-in, up-and-out, up-and-in)

**Greeks:** Delta, Gamma, Vega, Theta, Rho, all analytical (Black-Scholes), with full (S, T) surface plots

---

## Key Results

### Model Calibration (SPY, March 2026)

All three stochastic models were numerically calibrated to a live SPY option chain snapshot using multi-start L-BFGS-B optimization.

| Model | RMSE vs Market IV | Notes |
|-------|-------------------|-------|
| Heston | **1.93%** | 5-parameter global surface fit; k=4.86, theta=4.1%, xi=0.63, rho=-0.93 |
| SABR | **0.97%** | Per-slice calibration; 100% OOS pass rate |
| Merton | **5.88%** | Targeted short-term OTM puts; lambda=0.52 jumps/yr, mu_j=-8% |

Heston's strong leverage effect (rho = -0.93) and fast mean reversion (k = 4.86) capture SPY's pronounced downward IV skew. SABR achieves the tightest fit through per-expiry calibration. Merton best explains the steep short-term skew driven by jump risk.

### Numerical Methods

| Result | Value |
|--------|-------|
| Binomial tree error at N=500 steps | < 0.005 vs Black-Scholes |
| Monte Carlo std error at 100k paths | < 0.02 (within 1 SE of BS) |
| MC variance reduction (antithetic variates) | 74.9% |
| MC variance reduction (control variates) | 98.2% |
| Crank-Nicolson vs Implicit (same N) | 2.9x more accurate |
| CN with N=20 vs Implicit with N=5,000 | CN wins (second-order accuracy) |

### Validation

93.2% overall test pass rate (82/88 checks) across all phases. Key checks include:
- BS put-call parity residual < 1e-8
- IV round-trip error < 1e-7
- Barrier parity: Down-Out + Down-In = Vanilla (within 2 SE)
- All stochastic models reduce to Black-Scholes when stochastic parameters go to 0
- Heston CF vs Monte Carlo: within 2 SE at ATM

### Barrier Option Model Risk

Barrier option prices vary 10-27% across models at the same strike and expiry, highlighting the sensitivity of path-dependent payoffs to model choice.

---

## Project Structure

```
derivatives_pricing_engine/
|
+-- notebooks/
|   +-- phase_1_get_data.ipynb                   # SPY option chain, IV surface, data pipeline
|   +-- phase_2_black_scholes_greeks.ipynb       # BS formula, Greeks, IV inversion
|   +-- phase_3_binomial_monte_carlo.ipynb       # CRR tree, GBM MC, exotics
|   +-- phase_4_finite_difference_method.ipynb   # PDE methods, stability analysis
|   +-- phase_5_advanced_stochastic_models.ipynb # Heston, Merton, SABR
|   +-- phase_6_model_calibration.ipynb          # Market calibration
|   +-- phase_7_visualization.ipynb              # Final plots and analysis
|   +-- phase_8_OOS_validation.ipynb             # Out-of-sample validation suite
|
+-- data/
|   +-- SPY_options_<date>.csv           # Saved option chain snapshots
|   +-- SPY_meta_<date>.csv              # Market parameters (S, r, q)
|   +-- calibrated_params.json          # Saved model parameters
|
+-- outputs/                             # All generated plots (PNG)
+-- requirements.txt
+-- README.md
```

---

## Installation

```bash
git clone https://github.com/kshitijbhandari/Options-Pricing-and-Volatility-Surface-Modeling-in-Python.git
cd Options-Pricing-and-Volatility-Surface-Modeling-in-Python

python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

pip install -r requirements.txt
jupyter notebook
```

---

## Quickstart

Run phases in order; each notebook saves outputs that the next one loads:

```bash
# Phase 1: Fetches live SPY data (requires internet)
jupyter nbconvert --to notebook --execute notebooks/phase_1_get_data.ipynb

# Phases 2-8: Use saved data, can run anytime
jupyter nbconvert --to notebook --execute notebooks/phase_2_black_scholes_greeks.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_3_binomial_monte_carlo.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_4_finite_difference_method.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_5_advanced_stochastic_models.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_6_model_calibration.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_7_visualization.ipynb
jupyter nbconvert --to notebook --execute notebooks/phase_8_OOS_validation.ipynb
```

---

## Phase Overview

**Phase 1: Market Data Pipeline**
Fetches live SPY option chains via `yfinance`, filters by liquidity and moneyness, computes market implied volatilities, and saves a clean snapshot for all downstream phases. Put-call parity errors are checked per expiry and correlate 0.943 with time-to-expiry, confirming the deviations are a stale-quote artifact rather than a market inefficiency.

**Phase 2: Black-Scholes and Greeks**
Closed-form European option pricing with all five Greeks. IV inversion uses Newton-Raphson as the primary method with Brent's method as fallback for deep OTM/ITM options where vega approaches zero. Serves as the analytical benchmark validated throughout the project.

**Phase 3: Binomial Tree and Monte Carlo**
CRR binomial tree for European and American exercise, and Monte Carlo simulation using Euler-Maruyama discretization under the risk-neutral measure. Variance reduction via antithetic variates (74.9% reduction) and control variates (98.2% reduction). Extended to Asian and barrier exotic payoffs.

**Phase 4: Finite Difference Methods**
Solves the Black-Scholes PDE on a (S, t) grid using Explicit, Implicit, and Crank-Nicolson schemes. The tridiagonal system at each time step is solved using `scipy.linalg.solve_banded` (Thomas algorithm, O(N) per step). Demonstrates stability breakdown of the Explicit scheme and CN's second-order accuracy advantage over Implicit.

**Phase 5: Advanced Stochastic Models**
Heston stochastic volatility priced via the Lewis (2001) single-integral characteristic function formula, which avoids the branch-cut discontinuity present in the original Heston (1993) formulation for certain rho/xi parameter combinations. Merton jump diffusion priced via series expansion. SABR via Hagan et al. approximation. All three reproduce the downward IV skew that Black-Scholes cannot explain.

**Phase 6: Model Calibration**
Multi-start L-BFGS-B calibration of Heston (5 params, subsampled to 40 representative contracts from 960) and Merton (3 params, targeted at short-term OTM puts) to live market data. Calibration objective is liquidity-weighted: each contract weighted by 1/bid-ask spread so tight-spread contracts dominate the fit. SABR calibrated per expiry slice. Calibrated parameters stored in `data/calibrated_params.json`.

**Phase 7: Visualization**
Five publication-quality plots: IV surface comparison (2x2 grid), per-expiry smile overlays with RMSE annotations, Greeks surfaces over (S, T) space, convergence plots for binomial and MC, and ATM vol / IV skew term structure across all models.

**Phase 8: Out-of-Sample Validation**
End-to-end validation suite with 88 tests across all phases. 93.2% pass rate (82/88). Covers pricing correctness, convergence rates, model consistency, and exotic option pricing identities. Also quantifies barrier option model risk: prices vary 10-27% across models at the same strike and expiry, demonstrating why exotic products are sensitive to model choice.
