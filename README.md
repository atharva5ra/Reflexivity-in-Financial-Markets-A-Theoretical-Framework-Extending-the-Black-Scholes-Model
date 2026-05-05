# Reflexivity-in-Financial-Markets-A-Theoretical-Framework-Extending-the-Black-Scholes-Model
<br/>

"Financial markets are not passive mirrors of fundamentals — they are self-referential systems in which participants' perceptions and actions influence the very variables they use to predict the market."
— George Soros, The Alchemy of Finance

<br/>
</div>

Overview
Classical option pricing assumes market dynamics are exogenous — that the models we use to trade have no effect on the inputs we feed them. This assumption is violated every day by the synchronized behavior of institutional traders, hedge funds, and algorithmic systems all running variants of the same mathematical models.
This repository introduces a formal mathematical framework for reflexivity — the feedback loop between model-based trading strategies and the market variables those strategies depend on — integrated directly into the Black–Scholes stochastic differential equation.
<br/>
The Core Problem
Traditional Black–Scholes:
  dSt = μSt dt + σt St dWt
        ───────────────────
        Assumes Wt is purely exogenous

Reality:
  When α → 1 (most traders use the same model),
  Wt is no longer random — it is shaped by the model itself.
The more traders rely on a mathematical model, the more that model's assumptions cease to hold.
<br/>
Framework at a Glance
The reflexivity-extended SDE under the risk-neutral measure Q:
dSt=(r+βαt)St dt+σt(1−αt)+ε  St dWtQdS_t = \left(r + \beta\alpha_t\right) S_t \, dt + \sigma_t \sqrt{(1 - \alpha_t) + \varepsilon} \; S_t \, dW_t^QdSt​=(r+βαt​)St​dt+σt​(1−αt​)+ε​St​dWtQ​
with endogenous volatility:
σt2=σ02+λ αt−1 σt−12\sigma_t^2 = \sigma_0^2 + \lambda \, \alpha_{t-1} \, \sigma_{t-1}^2σt2​=σ02​+λαt−1​σt−12​
ParameterDescriptionαt ∈ [0,1]Proportion of market participants using model-based strategiesβReflexive drift amplification coefficientλVolatility feedback (endogenous amplification) strengthεIrreducible noise floor (prevents diffusion collapse)σ₀Exogenous baseline volatility
Three Mechanisms of Reflexivity
┌─────────────────────────────────────────────────────────────┐
│  1. PATH DEPENDENCE      Trader model use induces           │
│                          autocorrelation in Brownian        │
│                          increments, breaking the           │
│                          martingale property of Wt          │
├─────────────────────────────────────────────────────────────┤
│  2. DRIFT DEVIATION      Large-scale institutional use      │
│                          shifts drift away from the         │
│                          risk-free rate by βαt              │
├─────────────────────────────────────────────────────────────┤
│  3. VOLATILITY           σt depends on lagged volatility    │
│     ENDOGENEITY          and hedging intensity; feedback    │
│                          loops sustain elevated regimes     │
└─────────────────────────────────────────────────────────────┘
<br/>
Inferring α from Observable Greeks
Since αt is not directly observable, we infer it from option Greeks — quantities that reflect institutional hedging activity:
αt=11+e−zt,zt=ν0+ν1δ~t+ν2γ~t\alpha_t = \frac{1}{1 + e^{-z_t}}, \quad z_t = \nu_0 + \nu_1 \tilde{\delta}_t + \nu_2 \tilde{\gamma}_tαt​=1+e−zt​1​,zt​=ν0​+ν1​δ~t​+ν2​γ~​t​
where δ~t\tilde{\delta}_t
δ~t​ and γ~t\tilde{\gamma}_t
γ~​t​ are normalized moving averages of delta and gamma. High institutional delta-hedging → high γ → high αt.
<br/>
Empirical Validation
Simulations run across 6 equity assets using PyTorch with 100,000 Monte Carlo paths and 252 time steps (one trading year). Historical data sourced from Yahoo Finance (2020–2025).
Assets Studied
TickerTypeReflexivity RegimeSPYIndex ETFMacro-driven; weak reflexivityQQQIndex ETFMacro-driven; weak reflexivityAAPLBlue-chipIntermediateTSLAHigh-vol equityStrong institutional feedbackNVDAHigh-vol equityStrong institutional feedbackGMESentiment-drivenEpisodic; regime-dependent
Key Findings
1. Volatility Amplification

Realized volatility increases monotonically with both α and λ across all assets, consistent with self-reinforcing endogenous feedback cycles.

2. Volatility Clustering Divergence

SPY, QQQ, AAPL: clustering increases with α — synchronized hedging flows sustain elevated variance regimes.
TSLA, NVDA, GME: clustering decreases with α — sentiment-driven bursts are suppressed by diffusion attenuation.

3. TSLA Outperformance (α = 0.3)
          RMSE       MAE       MAPE
GBM:    107.57     99.69     48.81%
REF:     72.36     60.82     27.88%   ← REF wins by ~33%
Direct empirical validation: the REF model outperforms where reflexive dynamics are operationally present.
4. Stability Boundary at α = 0.6

Pearson correlation inverts sign for SPY (−0.968), QQQ (−0.920), and GME (−0.719) — a quantitative marker of the reflexivity stability boundary beyond which feedback destabilizes price dynamics.

5. Tail Risk Amplification

At moderate α, REF produces lower bulk volatility but more extreme downside VaR. Reflexive markets appear calm under typical conditions and are more dangerous in the tails.

Asset VaR Comparison (α = 0.3):
                GBM        REF
  SPY:        -23.5%     -29.9%
  QQQ:        -27.7%     -37.4%
  TSLA:       -78.5%    -118.7%
  NVDA:       -39.6%     -81.9%
<br/>
