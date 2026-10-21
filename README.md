# Abigail Ray || ECON 5200 || Diagnosing Broken Confidence Intervals

## Objective
I investigated how bootstrap confidence intervals can fail and compared methods for estimating uncertainty in discrete, skewed, clustered, and time-dependent data.

## Methodology
- Applied the ordinary bootstrap to the median age of 32,561 respondents and compared it with the bias-corrected and accelerated (BCa) interval.
- Simulated 400 samples of size 50 from a lognormal distribution and measured coverage for parametric and bootstrap intervals for the mean and median.
- Simulated wages for 1,000 workers across 25 firms and compared an i.i.d. bootstrap that resampled individual workers with a cluster bootstrap that resampled whole firms.
- Created and imported a `bootstrap_utils.py` module containing percentile, i.i.d. bootstrap, and cluster-bootstrap functions.
- Used an AI-generated coverage function, revised the prompt to specify the seed convention, and checked its results against the original coverage simulation.
- Simulated an AR(1) time series with 500 periods and compared i.i.d. resampling with a moving-block bootstrap using block length 20.

## Key Findings
- The bootstrap median age had a standard error of 0, and the BCa method returned `nan` with a warning because the bootstrap distribution was degenerate.
- For the simulated mean, the parametric interval had 92.5% coverage and the bootstrap interval had 92.0% coverage. For the median, the parametric interval had 66.8% coverage, while the bootstrap interval had 92.8% coverage.
- Resampling workers independently produced an interval 5.7 times narrower than the cluster-bootstrap interval. The naive interval covered the true mean in 19.5% of simulated panels, compared with 91.0% for the cluster bootstrap.
- The AI coverage function matched the original simulation exactly after the seed handling was revised: coverage was 19.5% for the naive interval and 91.0% for the clustered interval.
- In the AR(1) exercise, the i.i.d. bootstrap interval width was 0.287, compared with 0.600 for the moving-block bootstrap. Preserving dependence between nearby observations produced a wider interval.
