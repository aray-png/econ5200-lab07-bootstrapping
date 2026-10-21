# Lab 7: Diagnosing Broken Confidence Intervals

**Author:** Abigail Ray  
**Course:** ECON 5200: Causal Machine Learning & Applied Analytics  
**Repository:** `econ5200-lab07-bootstrapping`

## Project Objective

In this lab, I investigated why bootstrap confidence intervals can fail, even when the code runs correctly, and compared methods for estimating uncertainty with skewed data, clustered observations, and time-series dependence.

## Methodology

- **Diagnosis 1 — Degenerate Bootstrap:** I used the UCI Adult Census Income dataset, containing 32,561 respondents, to estimate the median age. I generated 2,000 bootstrap resamples and examined the bootstrap standard error, percentile confidence interval, and BCa interval.

- **Diagnosis 2 — Coverage Simulation:** I simulated 400 samples of 50 observations from a right-skewed lognormal distribution. I compared parametric and percentile bootstrap intervals for the mean and median, measuring how often each interval contained the true population value.

- **Diagnosis 3 — Clustered Data:** I simulated wages for 1,000 workers across 25 firms. I compared an ordinary bootstrap that resampled individual workers with a cluster bootstrap that resampled entire firms. I then measured the coverage of both methods across 200 simulated panels.

- **Python Module:** I used `src/bootstrap_utils.py` to organize the percentile confidence interval, ordinary bootstrap, and cluster bootstrap functions into an importable module.

- **AI-Assisted Coding:** I tested an AI-generated coverage function, revised the prompt to control seed handling, and compared the resulting coverage estimates against the results from Diagnosis 3.

- **Bayesian Bootstrap Extension:** I used Dirichlet-distributed weights to calculate Bayesian bootstrap estimates of the mean and compared them with frequentist bootstrap intervals.

- **Time-Series Challenge:** I simulated an AR(1) time series with a persistence parameter of 0.8 and 500 periods. I compared an ordinary bootstrap with a moving-block bootstrap using blocks of length 20.

## Key Findings

### 1. Degenerate Bootstrap

The sample median age was 37.0. Across 2,000 bootstrap resamples, the bootstrap standard error was 0, and the percentile confidence interval was [37.0, 37.0]. The BCa method returned `[nan, nan]` with warnings because the bootstrap distribution was degenerate.

This showed me that a zero bootstrap standard error does not mean the population median is known exactly. Instead, the repeated ties in the age data prevented the bootstrap from capturing uncertainty in the median.

### 2. Confidence Interval Coverage

The nominal confidence level was 95%, but the methods performed differently:

| Target | Method | Coverage | Monte Carlo SE |
|---|---|---:|---:|
| Mean | Parametric | 92.5% | 1.3% |
| Mean | Percentile bootstrap | 92.0% | 1.4% |
| Median | Parametric | 66.8% | 2.4% |
| Median | Percentile bootstrap | 92.8% | 1.3% |

The parametric and bootstrap intervals for the mean had similar coverage. The parametric interval performed poorly for the median because it used the sample mean and its standard error rather than an interval designed for the median.

The median parametric interval missed the true median in 33.2% of experiments, compared with the intended 5% miss rate. If these rates generalized to 1,000 decisions per year, that would mean approximately 282 additional misses per year.

### 3. Clustered Wages

Resampling workers independently produced a 95% confidence interval of $59,092–$60,121, with a width of $1,029. Resampling entire firms produced an interval of $56,624–$62,465, with a width of $5,841.

The cluster-bootstrap interval was approximately 5.7 times wider than the naive interval.

Across 200 simulated panels, the naive bootstrap achieved only 19.5% coverage, while the cluster bootstrap achieved 91.0%. This demonstrated how treating workers from the same firm as independent observations can seriously underestimate uncertainty.

### 4. AI-Assisted Coverage Function

The revised AI-generated function reproduced the original coverage results exactly: 19.5% for the naive bootstrap and 91.0% for the cluster bootstrap.

This exercise showed me that AI can help write reusable simulation functions, but I still need to check assumptions about random seeds and verify the results against known outputs.

### 5. Bayesian Bootstrap

The Bayesian bootstrap replaces ordinary resampling with randomly drawn Dirichlet weights. Under appropriate regularity conditions, Bayesian and frequentist bootstrap intervals can become asymptotically equivalent, although they may differ more in small samples or with skewed data.

Their interpretations are different: frequentist confidence intervals describe long-run coverage across repeated samples, while Bayesian credible intervals express posterior probability under the specified model.

### 6. Time-Series Block Bootstrap

For the simulated AR(1) series, the ordinary bootstrap interval had a width of approximately 0.287, while the moving-block bootstrap interval had a width of approximately 0.600.

The block bootstrap produced the wider interval because it preserved some of the serial dependence between neighboring observations. This showed me why resampling individual periods can underestimate uncertainty when observations are correlated over time.

## Conclusion

This lab taught me that a confidence interval can look reasonable while giving misleading results. I learned to examine bootstrap distributions, evaluate empirical coverage, account for dependence between observations, and verify simulation code rather than relying only on whether it runs successfully. The appropriate bootstrap method depends on the statistic being estimated and the structure of the data.

## Files

- `Copy of lab-ch07-diagnostic.ipynb` — Notebook containing the diagnoses, simulations, and extensions.
- `src/bootstrap_utils.py` — Reusable functions for percentile intervals and ordinary and cluster bootstrap procedures.
- `README.md` — Project overview, methodology, results, and conclusions.

## Reproducibility

The notebook uses NumPy random number generators with fixed seeds for reproducible simulations. Results should be checked against the outputs generated by the notebook before submission.
