# Mutual Fund Portfolio Optimization

**An uncertainty-aware review of my mutual fund investments**

## Overview

I hold a mutual fund portfolio of 17 funds accumulated through years of regular investments through Systematic Investment Plans (SIPs) and sporadic bulk investments (often associated with annual performance appraisals). Of late, I have been questioning my own rationale behind my investments which have been purely based on intuitions and not data. Through this portfolio piece, I wanted to review my current portfolio and identify if I am holding 17 differentiated funds or a smaller number dressed up as 17 and how far is my current allocation from an efficient one.

For the analysis, this project fetches historical NAV (Net Asset Value) data for all the 17 funds, calculates the return and risk and runs the mean-variance optimization using an optimizer built from scratch using scipy and validates the results against those from PyPortfolioOpt. The optimal allocation is also stress-tested to check its trustworthiness.

While the optimizer works fine and generates a clean answer, the subsequent analysis reveals that the generated answer shouldn't be followed naively.

## Key Findings

**Portfolio of 17 funds is reduced to 3** The maximum-Sharpe portfolio assigns weights only to three funds, confirming heavy redundancy across the holdings.
**My current portfolio allocation is inefficient** When I plotted the efficient frontier, the current portfolio sits well inside it with a Sharpe ratio of 0.49 against a ceiling of 0.87, which indicates that I have been taking more risks for less returns. The current portfolio is giving me a return of 13.6% at 13.3% volatility, while I could have achieved 17.8% return with 12.4% volatility.
**The redundancy among funds is not just noise, but significant** All the three small-cap funds, the two technology funds, and the large/mid-cap funds are heavily correlated.
**The optimal weights are highly unstable** Extending the estimation window from 3.75 years to 5 years change the optimal weights significantly. This uncertainty needs to be addressed / fixed.

## Methodology

1. **Data Layer:** The funds are filtered from AMFI scheme master using ISIN and historical NAV data is fetched from mfapi.in. A clean daily-returns matrix was built and weights were calculated in two estimation windows in parallel to test the window sensitivity.
2. **Statistics:** Annualized returns, volatility, and the full covariance and correlation matrices were calculated.
3. **Optimization:** Maximum-Sharpe and Minimum-Variance portfolios implemented from scratch using scipy.optimize and subsequently validated against PyPortfolioOpt.
4. **Critique Layer:** Bootstrapped resampling to quantify weight instability, a Sortino-based objective function that penalises downside risk alone, covariance shrinkage, an active-versus-index comparison, and a write-up on why the optimizer output should be treated with caution.

## Significance of the critique layer

Mean-variance optimization comes with its own shortcomings. It's based on the estimates of future returns alone and the output weights are highly sensitive to variations in the input data. While this project calculates those weights, the emphasis is on demonstrating how susceptible they are to changes and why an investor should ideally use the results to optimize the portfolio by consolidating the redundant funds or deinvesting those funds which have been underperforming consistently rather than blindly adopting the computed allocation.

I stress tested the reliability of the 'optimal' weights using bootstrapped resampling and realized that the weights were not numbers but rather a wide distribution. The resampling was done using 1000 samples and Parag Parikh's weight ranged from 0 to 100% during this process. For every significant fund (weight greater than 0), the standard deviation of the weights roughly equalled their respective means. 

I also ran a check with replacing Sharpe with Sortino as the risk measure. The optimizer returned the same three funds, but with different weights. This also proves that the definition of the 'optimal' portfolio depends on my risk appetite as well.

To ensure the method is fool-proof, I also tried Ledoit-Wolf shrinkage fix to reduce the instability. However, the results were largely unchanged. While shrinkage fixed the covariance noise, the instability mostly arises from the noisy return estimates which are not addressed by the shrinkage fix.

I also did an active-versus-index funds comparison as the last step and the result was a relief. The optimizer returned my active funds alone, an otherwise result would have made me question my entire investment strategy. However, this is by design as the comparison spoke only for the three funds picked by the optimizer and not my entire portfolio.

## Repository Structure

- `portfolio_optimization.ipynb` - complete analysis notebook
- `data/` - the holding portfolio (fund names, ISINs, portfolio slots, investment details not disclosed)
- `outputs/` - saved figures (correlation heatmaps, efficient frontier, allocation comparison)
- `requirements.txt` - dependencies

## Limitations

- Return estimates are based on historical data alone. Future results are not just a function of past data.
- A single risk-free-rate of 7% is assumed for the above analysis. This is approximately the Indian 10-year G-sec at the time of analysis.

**I wrote and understood every line of code in this repository, AI was used as a learning and debugging aid, not a code generator**
