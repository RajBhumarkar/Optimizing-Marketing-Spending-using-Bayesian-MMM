# Optimizing Marketing Spending using Bayesian MMM

## Project Overview

Can marketing spend be reallocated or reduced while maintaining projected GMV? This project models the relationship between weekly marketing activity and gross merchandise value, then uses the fitted response curves to compare budget choices.

The dataset is **synthetic**, with 156 weekly observations from January 2023 to December 2025. It includes paid search, social, display, video and affiliate spend, plus price, promotions and holidays. Channel efficiencies differ, advertising carries over across weeks, and responses flatten as spending grows. Trend, annual seasonality and random noise also affect GMV. All monetary values are illustrative currency units (CU).

The analysis lives in one sequential notebook. The CSV was generated once using seed 42; ordinary notebook execution loads it rather than generating new data. An optional, non-executing recipe in the notebook documents how to reproduce it. No optimization outcome was used to construct the data.

## Methodology

1. **Data preparation:** inspect weekly records and scale spend, GMV and controls transparently.
2. **Geometric adstock:** estimate channel-specific decay using the current week and seven previous weeks, with normalized weights and zero prehistory.
3. **Saturation response:** use a concave Hill curve with exponent 1 and a separate half-saturation point for each channel.
4. **Bayesian MMM in PyMC:** estimate baseline, media and control effects using regularizing priors; sample two chains with 1,000 tuning and 1,000 retained draws each.
5. **Channel contribution / response curves:** calculate attribution from posterior mean parameters and visualize response uncertainty.
6. **SLSQP budget optimization:** first maximize GMV at a fixed budget, then minimize spend while retaining at least 97% of current projected total GMV. Both scenarios bound each channel at 30–180% of current spend.

## Results

These are the executed notebook's **minimum-budget scenario** results, per week:

| Measure | Result |
|---|---:|
| Current Budget | 95,431.65 CU |
| Optimized Budget | 73,895.08 CU |
| Budget Reduction | 22.57% |
| Current Projected GMV | 355,636.18 CU |
| Optimized Projected GMV | 344,967.10 CU |

The fixed-budget scenario increases projected GMV to **367,606.58 CU**, an improvement of **3.37%**, with its largest spend increase in Affiliate. The minimum-budget scenario's largest absolute cut is in Display. Affiliate has the highest historical attributed GMV per spend, while Display has the strongest relative decline in marginal return versus near-zero spend.

Maximum R-hat is **1.0033**, minimum bulk/tail ESS are **749.0/880.4**, and there are **0 divergences**. In-sample MAE is **5,145.08 CU** and RMSE is **6,439.91 CU**. Both SLSQP runs report success; the notebook verifies budget equality, channel bounds and the GMV constraint numerically.

Only **65.1%** of posterior draws meet the relative 97% GMV target for the selected minimum budget. The point-estimate constraint is therefore not a high-confidence guarantee. Full tables, trace plots and constraint checks are retained in the notebook.

## Repository Structure

```text
Optimizing-Marketing-Spending/
├── data/
│   └── marketing_data.csv
├── outputs/
│   ├── optimized_budget.csv
│   ├── model_fit.png
│   ├── channel_contributions.png
│   └── response_curves.png
├── marketing_mix_model.ipynb
├── README.md
└── requirements.txt
```

## Tools Used

Python, pandas, NumPy, PyMC, PyTensor, ArviZ, SciPy and Matplotlib. Numba compiles the sampling calculations without a separate C++ compiler.

## Limitations

- Synthetic data demonstrates methodology rather than supporting a real company's recommendation.
- MMM estimates statistical attribution; it does not prove causality.
- Response curves depend on priors, additive effects and the fixed concave saturation shape. Prior sensitivity and time-based validation are not included.
- Optimization relies on estimated posterior response behavior. It assumes sustained spending and average historical controls, excluding transition effects and future observation noise.
- Historical contribution shares use observed GMV as the denominator. Estimated ROI is revenue ROAS, not profit ROI. The model-fit scores are in-sample.

## How to Run

Use Python 3.13 in a fresh virtual environment. From the repository root:

```bash
pip install -r requirements.txt
jupyter notebook marketing_mix_model.ipynb
```

Select **Restart Kernel and Run All**. Sampling takes several minutes depending on hardware; both chains run sequentially for portability. The notebook recreates the four output files. Package versions are pinned, and sampling uses seed 42; small numerical differences across platforms are possible.
