# Reflexivity-in-Financial-Markets-A-Theoretical-Framework-Extending-the-Black-Scholes-Model
<div align="center">

<br/>

<h1>Reflexivity in Financial Markets</h1>
<h3>A Theoretical Framework Extending the Black–Scholes Model</h3>

<br/>

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PyTorch](https://img.shields.io/badge/PyTorch-Monte_Carlo-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)](https://pytorch.org)

<br/>

*"Financial markets are not passive mirrors of fundamentals — they are self-referential systems<br/>in which participants' perceptions and actions influence the very variables they use to predict the market."*

**— George Soros, The Alchemy of Finance**

<br/>

</div>

---

## Overview

Classical option pricing assumes market dynamics are **exogenous** — that the models we use to trade have no effect on the inputs we feed them. This assumption is violated every day by the synchronized behavior of institutional traders, hedge funds, and algorithmic systems all running variants of the same mathematical models.

This repository introduces a formal mathematical framework for **reflexivity** — the feedback loop between model-based trading strategies and the market variables those strategies depend on — integrated directly into the Black–Scholes stochastic differential equation.

---

## The Core Problem

```
Classical Black–Scholes:

  dSt = μSt dt + σt St dWt
        ──────────────────────────────────────────
        Assumes Wt is purely exogenous (random)

Reality:

  When α → 1 (most traders use the same model),
  Wt is no longer random — it is shaped by the model itself.
  The more traders rely on a model, the more its assumptions cease to hold.
```

---

## Framework

### The Reflexivity-Extended SDE

Under the risk-neutral measure **Q**:

```
dSt = ( r + β·αt ) · St · dt  +  σt · √( (1 − αt) + ε ) · St · dWt
       ─────────────────────         ─────────────────────────────────
       Drift with reflexive           Diffusion attenuated by model
       feedback term βαt              adoption intensity αt
```

With **endogenous volatility**:

```
σt² = σ₀² + λ · αt₋₁ · σt₋₁²
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `αt ∈ [0, 1]` | Proportion of market participants using model-based strategies |
| `β` | Reflexive drift amplification coefficient |
| `λ` | Volatility feedback (endogenous amplification) strength |
| `ε` | Irreducible noise floor — prevents diffusion collapse as αt → 1 |
| `σ₀` | Exogenous baseline volatility at contract initiation |

### Three Mechanisms of Reflexivity

```
┌──────────────────────────────────────────────────────────────┐
│  1. PATH DEPENDENCE      Trader model use induces            │
│                          autocorrelation in Brownian         │
│                          increments, breaking the            │
│                          martingale property of Wt           │
├──────────────────────────────────────────────────────────────┤
│  2. DRIFT DEVIATION      Large-scale institutional use       │
│                          shifts drift away from the          │
│                          risk-free rate r by βαt             │
├──────────────────────────────────────────────────────────────┤
│  3. VOLATILITY           σt depends on lagged volatility     │
│     ENDOGENEITY          and hedging intensity; feedback     │
│                          loops sustain elevated regimes      │
└──────────────────────────────────────────────────────────────┘
```

### Inferring α from Observable Greeks

Since αt is not directly observable, it is inferred from **option Greeks** that reflect institutional hedging activity:

```
αt  =  1 / ( 1 + exp(−zt) )

zt  =  ν₀ + ν₁·δ̃t + ν₂·γ̃t

where  δ̃t = |δ|t / δmax     (normalized moving average of delta)
       γ̃t = |γ|t / γmax     (normalized moving average of gamma)
```

High institutional delta-hedging → high γ → high αt → stronger reflexive coupling.

---

## Empirical Validation

Simulations run across **6 equity assets** using PyTorch with **100,000 Monte Carlo paths** and **252 time steps** (one trading year). Historical data sourced from Yahoo Finance (2020–2025). A grid search over **17,000 parameter configurations** of (β, λ) was performed at each α level.

### Assets Studied

| Ticker | Type | Reflexivity Regime |
|--------|------|--------------------|
| SPY | Index ETF | Macro-driven — weak reflexivity |
| QQQ | Index ETF | Macro-driven — weak reflexivity |
| AAPL | Blue-chip equity | Intermediate |
| TSLA | High-vol equity | **Strong institutional feedback** |
| NVDA | High-vol equity | **Strong institutional feedback** |
| GME | Sentiment-driven equity | Episodic; regime-dependent |

### Key Findings

**1 — Volatility Amplification**

Realized volatility increases monotonically with both α and λ across all assets, consistent with the theoretical prediction that endogenous feedback creates self-reinforcing amplification cycles.

---

**2 — Volatility Clustering Divergence**

- **SPY, QQQ, AAPL**: clustering *increases* with α — synchronized institutional hedging flows endogenously sustain elevated variance regimes.
- **TSLA, NVDA, GME**: clustering *decreases* with α — sentiment-driven stochastic bursts are suppressed by diffusion attenuation at high α.

---

**3 — TSLA: Direct Empirical Validation (α = 0.3)**

```
              RMSE       MAE        MAPE
  ─────────────────────────────────────────
  GBM        107.57     99.69      48.81%
  REF         72.36     60.82      27.88%   ← REF outperforms 
  ─────────────────────────────────────────
```

The REF model outperforms GBM precisely where reflexive dynamics are most operationally present — assets dominated by institutional options hedging and gamma-driven feedback.

---

**4 — Stability Boundary at α = 0.6**

Pearson correlation against realized prices **inverts sign** for:

```
  SPY  →  −0.968      QQQ  →  −0.920      GME  →  −0.719
```

These sign inversions are not numerical artifacts. They mark a measurable regime boundary beyond which endogenous feedback generates paths that move *systematically opposite* to realized market trajectories — a quantitative marker of the reflexivity stability threshold.

---

**5 — Tail Risk Amplification**

At moderate α, REF produces lower bulk volatility but **more extreme downside VaR**. Reflexive markets appear stable under typical conditions and are more dangerous in the tails.

```
  Asset VaR at α = 0.3:
  ───────────────────────────────────
  Asset      GBM VaR      REF VaR
  ───────────────────────────────────
  SPY        −23.5%       −29.9%
  QQQ        −27.7%       −37.4%
  TSLA       −78.5%      −118.7%
  NVDA       −39.6%       −81.9%
  GME       −273.3%      −315.4%
  ───────────────────────────────────
```

---

## Repository Structure

```
.
.
├── notebooks/
│   └── experiment.ipynb
│       # Core notebook: model implementation, simulation, and analysis

├── results/
│   ├── alpha_0.1/
│   ├── alpha_0.3/
│   ├── alpha_0.6/
│   ├── alpha_0.9/
│   │   ├── *.png
│   │   │   # Asset-wise comparisons (Real vs GBM vs Reflexive)
│   │   │   # Metric plots: RMSE, MAPE, VOL, CORR, KURT
│   │   ├── empirical_validation_results.csv
│   │   ├── model_winner_summary.csv
│   │   └── *.docx
│   │       # Per-alpha summary reports
│   │
│   └── primary/
│       ├── * Phase Diagram.png
│       ├── * Vol Clustering.png
│       ├── GBM Vs Ref Terminal Distributions.png
│       ├── Ref Monte Carlo Simulation.png
│       ├── reflexive_experiment_results.csv
│       ├── summary_statistics.csv
│       └── *.docx
│           # Primary experiment report and figures

└── README.md
```

---

## Requirements

```
python     >= 3.10
torch      >= 2.0
numpy      >= 1.24
pandas     >= 2.0
yfinance   >= 0.2
matplotlib >= 3.7
scipy      >= 1.10
```

---

## Theoretical Contributions

**1. First formal mathematical integration** of Soros-style reflexivity into the Black–Scholes SDE, moving beyond qualitative description to a calibratable, quantitative framework.

**2. Endogenous volatility operator** — the modular reflexivity term `λ · αt₋₁ · σt₋₁²` is structurally compatible with Heston, Hull-White, jump-diffusion, and GARCH frameworks.

**3. Stability boundary characterization** — correlation sign-inversions at α = 0.6 provide a measurable threshold for when reflexive feedback destabilizes market dynamics, with direct implications for regulatory circuit breaker design.

---

## Limitations & Future Work

| Area | Status |
|------|--------|
| Fat tails / kurtosis | Jump-diffusion extension needed; both GBM and REF yield near-zero kurtosis |
| Parameter calibration | Grid search → MLE/GMM from high-frequency options data |
| Closed-form pricing | Tractable case (constant α, β = 0) yields σ_eff = √(σ₀² + λ·αt₋₁·σt₋₁²), nested within Black–Scholes as α → 0 |
| Stability analysis | Formal Lyapunov characterization of critical (α, β, λ) thresholds |

---

## Citation

```bibtex
@inproceedings{anonymized2026reflexivity,
  title     = {Reflexivity in Financial Markets: A Theoretical Framework
               Extending the {Black--Scholes} Model},
  author    = {Anonymized for Review},
  booktitle = {Proceedings of the IEEE Conference on Computational
               Intelligence for Financial Engineering and Economics (CIFEr)},
  year      = {2026},
  note      = {Under Review}
}
```

---

## References

| | |
|--|--|
| [1] | Black & Scholes (1973). *The Pricing of Options and Corporate Liabilities.* Journal of Political Economy, 81(3), 637–654. |
| [2] | Merton (1973). *Theory of Rational Option Pricing.* Bell Journal of Economics, 4(1), 141–183. |
| [3] | Mandelbrot (1963). *The Variation of Certain Speculative Prices.* Journal of Business, 36(4), 394–419. |
| [4] | Cont (2001). *Empirical Properties of Asset Returns: Stylized Facts and Statistical Issues.* Quantitative Finance, 1(2), 223–236. |
| [5] | Brunnermeier & Sannikov (2016). *The I Theory of Money.* NBER Working Paper No. 22533. |
| [6] | Soros (2003). *The Alchemy of Finance.* Wiley. |
| [7] | Wiener (1923). *Differential-Space.* Journal of Mathematics and Physics, 2, 131–174. |
| [8] | Heston (1993). *A Closed-Form Solution for Options with Stochastic Volatility.* Review of Financial Studies, 6(2), 327–343. |
| [9] | Hull & White (1987). *The Pricing of Options on Assets with Stochastic Volatilities.* Journal of Finance, 42(2), 281–300. |
| [10] | Dupire (1994). *Pricing with a Smile.* Risk Magazine, 7(1), 18–20. |
| [11] | Merton (1976). *Option Pricing When Underlying Stock Returns Are Discontinuous.* Journal of Financial Economics, 3(1–2), 125–144. |
| [12] | Kou (2002). *A Jump-Diffusion Model for Option Pricing.* Management Science, 48(8), 1086–1101. |

---

<div align="center">
<sub>Submitted to IEEE CIFER 2026 — Identity anonymized for double-blind review</sub>
</div>
