
# Markowitz · Black–Litterman · Quadratic Transaction Costs

This repo is intentionally simple: a single **Jupyter notebook** implements all models end-to-end, and a `data/` folder holds inputs and (optionally) outputs. No extra Python modules—everything runs from the notebook.


---

## Theoretical Background

### 1. Markowitz Mean–Variance Optimization

The goal is to select portfolio weights $w$ that balance expected return and risk.

- Expected portfolio return:
$$\mu_p = w^\top \mu$$

- Portfolio variance:
$$\sigma_p^2 = w^\top \Sigma w$$

#### Problems of interest
- **Minimum variance:**
$$\min_w \; w^\top \Sigma w \quad \text{s.t.} \; \mathbf{1}^\top w = 1,\; w \ge 0 \text{ (if long-only)}$$

- **Maximum Sharpe ratio:**
$$\max_w \; \frac{w^\top \mu - r_f}{\sqrt{w^\top \Sigma w}} \quad \text{s.t.} \; \mathbf{1}^\top w = 1$$

- **Target return:**
$$\min_w \; w^\top \Sigma w \quad \text{s.t.} \; w^\top \mu \ge \mu^*,\; \mathbf{1}^\top w = 1$$

---

### 2. Black–Litterman Model

The BL model blends equilibrium returns with investor views.

- **Implied equilibrium returns:**
$$\pi = \delta \Sigma w_m$$
where $\delta$ is risk aversion, $\Sigma$ the covariance, and $w_m$ the market-cap weights.

- **Views formulation:**
$$P \mu = Q$$
where $P$ is a pick matrix selecting assets and $Q$ are view returns (absolute or relative).

- **Posterior distribution of returns:**

$$
\mu_{BL} = \left[(\tau \Sigma)^{-1} + P^\top \Omega^{-1} P \right]^{-1}
\left[(\tau \Sigma)^{-1} \pi + P^\top \Omega^{-1} Q \right]
$$

where $\tau$ is a scaling factor and $\Omega$ the view-uncertainty covariance (often diagonal).

- The posterior covariance is also adjusted:
$$\Sigma_{BL} = \Sigma + \left[(\tau \Sigma)^{-1} + P^\top \Omega^{-1} P \right]^{-1}$$

---

### 3. Quadratic Transaction Costs (QTC)

When rebalancing from current weights $w_0$ to new weights $w$, trading costs matter.

- Let trades be $x = w - w_0$.  
- Transaction costs are modeled as quadratic:
$$c(x) = \tfrac{1}{2} x^\top \Lambda x$$
with diagonal $\Lambda$ encoding per-asset liquidity/impact.

#### Optimization with QTC

$$
\max_x \; \lambda_\mu \, \mu^\top (w_0 + x)
$$

- $\tfrac{\lambda_r}{2} (w_0 + x)^\top \Sigma (w_0 + x)$
- $\tfrac{\lambda_c}{2} x^\top \Lambda x$

subject to the budget constraint $\mathbf{1}^\top (w_0 + x) = 1$ and optional bounds on turnover or per-asset trades.  
This is a convex quadratic program.

---

## What’s Inside (in the notebook)

- **Markowitz Mean–Variance**
  - Minimum-variance, Maximum-Sharpe, Target-return
  - Efficient frontier sampling & plotting
  - Long-only or bounded weights / optional shorts

- **Black–Litterman**
  - Reverse optimization to implied returns from benchmark weights
  - View blending via $\tau$ and $\Omega$ (absolute & relative views)
  - Run Markowitz on BL posterior mean

- **Quadratic Transaction Costs (QTC)**
  - One-step rebalance given current weights with quadratic trading costs
  - Turnover limits, per-asset trade bounds, and weight bounds

- **Utilities**
  - Prices → returns, annualization (daily/weekly/monthly)
  - CSV I/O for inputs and weights/frontier outputs
  - Publication-quality Matplotlib plots

---

## How to Use the Notebook

The notebook is organized into the following **sections** (cells):

1. **Imports & Config**
   - Set random seed, frequency (`daily/weekly/monthly`), risk-free rate, plotting style
2. **Load Data**
   - Read `data/prices.csv` → returns (simple or log), compute $\mu$ and $\Sigma$
   - Annualize using factor $\{252, 52, 12\}$
3. **Markowitz Functions**
   - `min_variance`, `max_sharpe`, `target_return`, `efficient_frontier`
4. **Run Markowitz Examples**
   - Print weights & stats; plot efficient frontier + CML
5. **Black–Litterman**
   - Parse `benchmark.csv` & `views.csv` → build $\pi, P, Q, \Omega$
   - Compute $\mu_{BL}$, re-run Markowitz on posterior mean
6. **Quadratic Transaction Costs**
   - Define current weights $w_0$, cost diag $\Lambda$, multipliers $\lambda_r, \lambda_\mu, \lambda_c$
   - Solve convex QP for trades $x$, output new weights $w = w_0 + x$
7. **Save Outputs**
   - Export weights (`out/*.csv`), frontier points, and plots (`out/*.png`)

Each section is self-contained; you can run only what you need.

---

## Typical Workflow

### A) Markowitz — Minimum Variance & Frontier
1. Point the notebook to `data/prices.csv`.
2. Set `long_only=True` and (optionally) bounds like `wmax=0.6`.
3. Run `min_variance` and `efficient_frontier(n_points=50)`.
4. Plot & export results.

### B) Black–Litterman — Views on Top of a Benchmark
1. Provide `data/benchmark.csv` (market weights).
2. Write your views in `data/views.csv`.
3. Pick `tau` (e.g., 0.05) and `risk_aversion` (e.g., 3.0).
4. Compute posterior mean $\mu_{BL}$, then run Max-Sharpe or desired target-return.

### C) Quadratic Transaction Costs — Rebalancing
1. Define $w_0$ (current weights) inline or load from CSV.
2. Choose trading-cost diagonal (e.g., `cost_diag = [0.002, ...]`).
3. Set multipliers: $\lambda_r$ , $\lambda_\mu$, $\lambda_c$.
4. Add turnover / per-asset trade bounds if needed and solve; export new weights.

---
