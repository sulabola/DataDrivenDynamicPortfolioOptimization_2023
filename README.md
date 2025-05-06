# High Frequency Data-Driven Dynamic Portfolio Optimization for Cryptocurrencies

Recently, there has been a growing interest in constructing portfolios with stocks and cryptocurrencies. As cryptocurrency prices increase over the years, there is a growing interest in investing in cryptocurrencies, along with diversifying portfolios by adding multiple cryptocurrencies to existing portfolios. Even though investing in cryptocurrency leads to high returns, it also leads to high risk due to the high uncertainty of cryptocurrency price changes. Thus, more robust risk measures have been introduced to capture market risk and avoid investment loss, along with different types of portfolios to mitigate risks. For most cryptocurrency returns, significant autocorrelation of the absolute value of returns is observed, suggesting volatility clustering. In order to incorporate volatility clustering, the data-driven exponentially weighted moving average (DDEWMA) covariance model is introduced in portfolio optimization.

The PDF copy of the paper can be downloaded from here: [Download Paper](https://ieeexplore.ieee.org/abstract/document/10371936) 

A preprint version of the paper is available in the repository.

Programming Language: [R](https://cran.r-project.org/bin/windows/base/) / [RStudio](https://posit.co/downloads/)

Data: The high-frequency cryptocurrency price data and treasury bill rates used are available in the CSV file in the repository. The following are the sources of data:
1. Cryptocurrency Price Data: [Yahoo!Finance](https://ca.finance.yahoo.com/)
2. Market Cap (current Price x circulating supply): [Coinmarketcap](http://www.coinmarketcap.com)
3. Treasury Bill Rates: [Bloomberg](https://www.bloomberg.com/canada)

### Methodology

Let the training period window for the study be $[1, T_1]$. If the last data point is collected at time $T_2$, the test period is denoted by $[T_1 + 1, T_2]$. After establishing the optimal smoothing constant through the training data, it will then be applied to predict volatility during the testing period. The DDEWMA algorithm for computing a one-step-ahead forecast of volatility and residuals is summarized below.

#### Algorithm 1: DDEWMA Volatility Forecasts of Returns

Input: Adjusted Closing Price of assets $P_{i, t}, t = 0, \ldots, k, \ldots T_1, \ldots T_2, i=1, \ldots, n$

1. Compute log returns:
```math
   R_{i,t} = \frac{P_{i, t} - P_{i, t-1}}{P_{i, t-1}}, \quad t = 1, \ldots, T_1, \ldots, T_2
```

2. Estimate $sign$ correlation:
```math
   \hat{\rho}_{i} = \text{Corr}(R_{i,t} - \bar{R}_{i}, \text{sign}(R_{i,t} - \bar{R}_{i}))
```

3. Compute normalized absolute residuals:
```math
   Z_{i,t} = \frac{|R_{i,t} - \bar{R}_{i}|}{\hat{\rho}_{i}}
```

4. Initialize smoothed series:
```math
   S_{i,0} = \frac{1}{k} \sum_{t=1}^{k} Z_{i,t}
```

5. Smoothing parameter:
```math
   \alpha_i \in (0, 1)
```

6. Apply EWMA to the training period:
```math
   S_{i,t} = \alpha Z_{i,t} + (1 - \alpha) S_{i,t - 1}, \quad t = 1, \ldots, T_1
```

7. Optimize the smoothing parameter by minimizing the one-step-ahead forecast error sum of squares:
```math
   \alpha_{i,\text{opt}} = \arg\min_{\alpha} \sum_{t = k+1}^{T_1} (Z_{i,t} - S_{i,t-1})^2
```

8. Apply optimal smoothing to the test period:
```math
   S_{i,t} = \alpha_{i,\text{opt}} Z_{i,t} + (1 - \alpha_{i,\text{opt}}) S_{i,t - 1}, \quad t = T_1+1, \ldots, T_2
```

9. Calculate standardized residuals:
```math
   \text{res}_{i,t} = \frac{R_{i,t} - \bar{R}_{i}}{S_{i,t - 1}}, \quad t = T_1+1, \ldots, T_2
```

Output:
  1. Volatility forecasts: $S_{i,t}, t=T_1, \ldots T_2$
  2. Standardized residuals: $res_{i,t}, t=1, 2, \ldots T_2$

The correlation matrix is generated based on the covariance matrix of the standardized residuals. Thus, an element in the data-driven covariance matrix can be written as
```math
\Sigma^{dd}[t-1]_{i, j} \gets cor^{dd}[t-1]_{i, j} \hat{\sigma}_{i, t}\hat{\sigma}_{j, t}.
```
Then, optimal weights for the portfolio are obtained using a data-driven covariance matrix as shown in Algorithm 2.

#### Algorithm 2: Portfolio Weights from DDEWMA Covariance Matrix

Input: Adjusted Closing Price of assets $P_{i, t}, \quad t = 0, \ldots, k, \ldots, T_1, \ldots, T_2, \quad i = 1, \ldots, n$

1. Compute returns:

```math
R_{i,t} = \frac{P_{i,t} - P_{i,t-1}}{P_{i,t-1}}, \quad t = 1, \ldots, T_2
```

2. Use the volatility forecast from Algorithm 1:

```math
\hat{\sigma}_{i,t} = S_{i,t-1}, \quad t = T_1+1, \ldots, T_2
```

3. Construct the diagonal of the covariance matrix:

```math
\Sigma^{dd}[t-1]_{i,i} = \hat{\sigma}_{i,t}^2
```

4. Construct the off-diagonal elements:

```math
\Sigma^{dd}[t-1]_{i,j} = cor^{dd}[t-1]_{i,j} \cdot \hat{\sigma}_{i,t} \cdot \hat{\sigma}_{j,t}
```

5. Ensure the matrix \$\Sigma^{dd}\[t-1]\$ is positive definite.

6. Compute optimal portfolio weights under mean-variance utility:

```math
w^{dd}_{t-1} = \frac{(\Sigma^{dd}[t-1])^{-1} (\mu_p - r_f)}{\mathbf{1}^\top (\Sigma^{dd}[t-1])^{-1} (\mu_p - r_f)}
```

Output:
Portfolio weights at time \$t-1\$: $w^{dd}_{t-1}$

### Findings

Experimental findings demonstrate that when constructing portfolios with cryptocurrencies, the DDEWMA covariance model surpasses the commonly used empirical covariance model. This superiority is evident through higher Sharpe ratios and reduced risk forecasts.

### References

1. Zhu, Z., Thavaneswaran, A., Paseka, A., Frank, J., \& Thulasiram, R. (2020, July). Portfolio optimization using a novel data-driven EWMA covariance model with big data. In 2020 IEEE 44th Annual Computers, Software, and Applications Conference (COMPSAC) (pp. 1308-1313). IEEE.
2. Thavaneswaran, A., Paseka, A., \& Frank, J. (2020). Generalized value at risk forecasting. Communications in Statistics-Theory and Methods, 49(20), 4988-4995.

