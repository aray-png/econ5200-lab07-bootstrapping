# ECON 5200 Lab 7: Diagnosing Broken Confidence Intervals

**Author:** Abigail Ray  
**Course:** ECON 5200 — Causal Machine Learning & Applied Analytics  
**Project:** Estimation & Bootstrapping  
**Repository:** `econ5200-lab07-bootstrapping`

---

## Project Overview

In this lab, I investigated how confidence intervals can produce misleading results when the statistical method does not match the characteristics of the data. I used the UCI Adult Census Income dataset and simulated economic data to examine degenerate bootstrap distributions, confidence interval coverage, clustered observations, and serial dependence in time-series data.

I also developed and tested an importable Python module containing bootstrap functions and evaluated an AI-generated simulation function against independently measured results.

## Learning Objectives

Through this project, I worked to:

- Diagnose bootstrap confidence intervals that fail silently or produce undefined results.
- Measure the empirical coverage of nominal 95% confidence intervals.
- Compare parametric and bootstrap intervals for means and medians.
- Identify how treating clustered observations as independent understates uncertainty.
- Implement and verify a cluster bootstrap procedure.
- Create reusable Python functions in an importable module.
- Evaluate AI-generated code by comparing its output against known simulation results.
- Explore Bayesian bootstrap methods and block bootstrapping for dependent time-series observations.

## Data and Tools

### Datasets

- **UCI Adult Census Income:** 32,561 respondents, including age and capital-gain variables.
- **Simulated lognormal data:** 400 experiments, each with a sample size of 50, used to measure confidence interval coverage.
- **Simulated clustered wage data:** 1,000 workers distributed across 25 firms, with a known population mean wage of $60,000.
- **Simulated AR(1) time series:** 500 observations with an autoregressive coefficient of 0.8.

### Tools and Libraries

- Python
- NumPy
- pandas
- SciPy
- `pathlib`
- `warnings`
- `importlib`

## Methodology

### 1. Diagnosing a Degenerate Bootstrap

I used the Adult Census dataset to estimate the median age and generated 2,000 bootstrap resamples by sampling observations with replacement.

I evaluated the bootstrap standard error, percentile confidence interval, and number of distinct bootstrap medians. I then compared the percentile interval with SciPy's bias-corrected and accelerated (BCa) method.

This analysis tested whether a confidence interval with zero width necessarily indicates that the population parameter is known exactly.

### 2. Measuring Confidence Interval Coverage

I simulated 400 samples of 50 observations from a right-skewed lognormal distribution. For each sample, I constructed confidence intervals for the mean and median using parametric and bootstrap methods.

I compared the empirical coverage rates with the nominal 95% confidence level and calculated Monte Carlo standard errors to evaluate sampling variability in the simulated results.

I also examined why a confidence interval designed for the mean can fail when evaluated against the population median.

### 3. Correcting for Clustered Observations

I simulated wages for 1,000 workers across 25 firms. Each firm's workers shared a firm-level wage effect, creating dependence between workers in the same firm.

I first calculated a naive bootstrap interval by independently resampling individual workers. I then implemented a cluster bootstrap that resampled entire firms while keeping the workers within each selected firm together.

Finally, I repeated the experiment across 200 simulated panels to compare the empirical coverage of the naive and clustered confidence intervals.

### 4. Creating an Importable Bootstrap Module

I organized the repaired procedures into `src/bootstrap_utils.py` so that the functions could be reused in subsequent analyses.

The module includes:

- `percentile_ci()`: Calculates a percentile confidence interval from bootstrap replicates.
- `bootstrap_mean()`: Generates ordinary i.i.d. bootstrap replicates of the mean.
- `cluster_bootstrap_mean()`: Generates bootstrap replicates by resampling complete clusters.

I imported the module into the notebook and checked that the cluster-bootstrap function reproduced an interval consistent with the original implementation.

### 5. Evaluating AI-Generated Code

I asked an AI system to write a reusable coverage function that repeatedly simulates datasets, constructs confidence intervals, and calculates the proportion containing the true population parameter.

I revised the prompt to specify the seed convention needed to reproduce the original simulation. I then compared the AI function's results with the coverage rates measured in the clustered-wage experiment.

This process demonstrated why generated code should be tested against known results rather than accepted solely because it runs successfully.

### 6. Exploring the Bayesian Bootstrap

I implemented a Bayesian bootstrap for the mean using weights drawn from a Dirichlet distribution.

Instead of resampling observations, the procedure assigns random weights to the observed values and calculates weighted means. I examined how the Bayesian and frequentist bootstrap intervals relate under suitable asymptotic conditions.

### 7. Examining Dependence in Time-Series Data

I simulated an AR(1) time series with 500 observations and an autoregressive coefficient of 0.8.

I compared two approaches for estimating uncertainty around the series mean:

- An i.i.d. bootstrap that independently resamples individual observations.
- A moving-block bootstrap that resamples contiguous blocks of 20 observations.

I compared the resulting interval widths to investigate how preserving serial dependence affects estimated uncertainty.

## Key Findings

### Finding 1: A Zero-Width Interval Does Not Establish Certainty

| Statistic | Result |
|---|---:|
| Number of respondents | 32,561 |
| Sample median age | 37.0 |
| Bootstrap resamples | 2,000 |
| Bootstrap standard error | 0.000000 |
| Percentile 95% confidence interval | [37.0, 37.0] |
| Distinct bootstrap medians | 1 |

The bootstrap median remained at 37.0 across all 2,000 resamples. The age variable contains many tied, discrete values, which caused the bootstrap distribution of the median to become degenerate.

The BCa procedure returned `[nan, nan]` and raised warnings because its acceleration calculation could not be evaluated reliably. I concluded that the zero-width interval reflected a limitation of the bootstrap procedure in this setting, rather than proof that the population median was known exactly.

### Finding 2: Confidence Interval Coverage Depends on the Statistic and Method

The lognormal simulation produced the following empirical coverage rates:

| Target | Method | Empirical coverage | Monte Carlo SE |
|---|---|---:|---:|
| Mean | Parametric | 92.5% | 1.3% |
| Mean | Bootstrap | 92.0% | 1.4% |
| Median | Parametric | 66.8% | 2.4% |
| Median | Bootstrap | 92.8% | 1.3% |

The parametric and bootstrap intervals for the mean had similar coverage rates, although both were below the nominal 95% level in this simulation.

The parametric interval performed much worse for the median because it reused an interval centered on the sample mean rather than constructing an interval appropriate for the median. For right-skewed data, the mean generally exceeds the median, which can shift the interval away from the true median.

The results showed that a method must match the parameter being estimated. A sample size greater than 30 does not, by itself, guarantee correct coverage for every statistic or confidence interval.

### Finding 3: Ignoring Firm-Level Dependence Understates Uncertainty

The simulated wage panel contained 1,000 workers across 25 firms, with a true mean wage of $60,000.

| Measure | Naive i.i.d. bootstrap | Cluster bootstrap |
|---|---:|---:|
| Lower confidence limit | $59,092 | $56,624 |
| Upper confidence limit | $60,121 | $62,465 |
| Interval width | $1,029 | $5,841 |
| Coverage over 200 panels | 19.5% | 91.0% |

The naive interval was approximately **5.7 times narrower** than the cluster-bootstrap interval. Although the naive interval contained the true mean in the original sample, it covered the true mean in only 19.5% of the 200 simulated panels.

The cluster bootstrap improved coverage to 91.0% by preserving the dependence between workers employed at the same firm. The results demonstrated that resampling individual observations can substantially understate uncertainty when the data are clustered.

### Finding 4: Reusable Code Requires Verification

I created and imported `src/bootstrap_utils.py`, which separates percentile interval calculations, ordinary bootstrap sampling, and cluster-bootstrap sampling into reusable functions.

The AI-generated coverage function reproduced the original simulation results after I revised the seed convention:

| Interval method | AI coverage | Original coverage |
|---|---:|---:|
| Naive bootstrap | 19.5% | 19.5% |
| Cluster bootstrap | 91.0% | 91.0% |

The exact agreement showed that the revised function reproduced the original simulation under the tested seed convention. It also demonstrated that a small implementation detail, such as how random seeds are passed between functions, can affect whether a simulation is directly comparable with previous results.

### Finding 5: The Bayesian Bootstrap Uses a Different Weighting Approach

The Bayesian bootstrap assigns random Dirichlet weights to observed values instead of repeatedly sampling observations with replacement.

Under suitable regularity conditions, Bayesian and frequentist bootstrap intervals for the mean become asymptotically equivalent. Their results may differ more noticeably in smaller samples, particularly when the data are skewed or contain influential observations.

The Bayesian interpretation also differs from the frequentist interpretation: a frequentist confidence interval describes long-run coverage across repeated samples, whereas a Bayesian credible interval describes posterior uncertainty conditional on the observed data and the specified model.

### Finding 6: Preserving Time Dependence Produces Wider Intervals

The AR(1) simulation produced the following interval widths:

| Method | 95% confidence interval | Interval width |
|---|---|---:|
| i.i.d. bootstrap | [-0.183, 0.104] | 0.287 |
| Moving-block bootstrap | [-0.323, 0.277] | 0.600 |

The moving-block bootstrap produced the wider interval because it preserved the dependence between nearby observations. Independently resampling individual periods breaks the original time-series structure and can understate uncertainty.

The block length creates a bias-variance tradeoff. Blocks that are too short may fail to preserve enough serial dependence, while blocks that are too long leave fewer effectively independent blocks for estimating uncertainty.

## Technical Deliverables

The project includes the following deliverables:

- **`Copy of lab-ch07-diagnostic.ipynb`** — The notebook containing the simulation experiments, diagnostic questions, code outputs, and interpretation.
- **`src/bootstrap_utils.py`** — The reusable Python module containing percentile, i.i.d. bootstrap, and cluster-bootstrap functions.
- **`README.md`** — This documentation of the project's objectives, methodology, results, and conclusions.

## Limitations and Considerations

- The coverage results come from simulations with specific distributional assumptions and sample sizes. They should not automatically be generalized to every real-world dataset.
- Empirical coverage estimates vary across simulations, so Monte Carlo uncertainty matters when comparing methods.
- The cluster-bootstrap experiment uses 25 firms, meaning the number of independent clusters is much smaller than the number of workers.
- A wider confidence interval is not automatically a correct confidence interval; repeated-sample coverage provides a more informative diagnostic.
- The time-series experiment uses a specific AR(1) process and block length. Different dependence structures and block lengths may produce different results.
- The AI-generated function was checked against a specific simulation and seed convention. Its agreement in this experiment does not guarantee correctness for every application.

## Conclusion

This lab showed me that a confidence interval can look precise and still be misleading. A degenerate bootstrap can hide uncertainty, an interval constructed for the wrong statistic can under-cover, and treating dependent observations as independent can produce severely understated standard errors.

I also learned the importance of validating simulation code against known results and structuring statistical functions so they can be reused and tested. Across the exercises, the central lesson was that reliable inference depends not only on calculating an interval correctly, but also on choosing a method that reflects the data's structure and the parameter being estimated.

---

## Reproducibility

The notebook uses NumPy random generators with explicit seeds for reproducibility. The reported numerical results correspond to the outputs recorded in the notebook.

**Repository:** `econ5200-lab07-bootstrapping`
