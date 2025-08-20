---
title: "[Summary] Adding Error Bars to Evals: A Statistical Approach to Language Model Evaluations"
date: 2025-08-20
tags:
- Language Model Evaluation
- Error Bars
- Statistical Analysis
- Uncertainty Quantification
draft: false
---

## TL;DR
Machine learning model evaluation commonly reports the “highest number” often lacking any kind of statistical significance.
This creates misleading comparisons, especially when differences between models are small.
The paper overviews the different methods of adding statistical error bars to evals, covering independent and clustered questions, paired model comparisons, and power analysis. These tools help quantify uncertainty and avoid overconfident claims about which model is better.

## Motivation
LLM evals often treat the top score as definitive. But what if two models differ by less than one percentage point? Should we conclude one is stronger, or could random variation explain the margin?
Without statistical treatment, we risk drawing unwarranted conclusions especially in the current landscape of leaderboard reporting, where models are ranked by small decimal-level differences, and uncertainty is rarely mentioned. This work aims to align LLM evaluation with the rigor already standard in other scientific domains.

The paper describe the following insights:
1. Computing standard errors of the mean using the Central Limit Theorem is usually enough and no need for complicated bootstrap algorithms 
2. When questions are drawn in related groups, computing clustered standard errors should be used to not have too narrow error bars 
3. When two models are being compared, variance can be reduced by conducting statistical inference on the question-level paired differences, rather than population-level
4. Using power analysis to determine whether an eval (or a random subsample) is capable of testing a hypothesis of interest

## Calculating Error Bars of Independent questions
For independent test items, the standard error (SE) of the mean score can be estimated directly from the Central Limit Theorem:
$$
SE_{CLT} = \sqrt{\frac{1}{n(n-1)} \sum_{i=1}^n (s_i - \bar{s})^2 }
$$
where $s_i$ are the model scores and $\bar{s}$ their mean.
In the special case when $s_i \in \{0,1\}$ (binary outcomes),
$$
SE_{Bernoulli} = \sqrt{\frac{\bar{s}(1-\bar{s})}{n}}.
$$
Notice that this Bernoulli form is incorrect if scores are fractional (e.g., F1), where it tends to overestimate the interval. 

To calculate a 95% confidence interval is then
$$
CI_{95\%} = \bar{s} \pm 1.96 \times SE.
$$


### Numerical examples
Assume $n=200$ items, binary scoring $s_i \in \{0,1\}$, where 1 = correct, 0 = incorrect.


The SE depends on the model’s observed accuracy ($\bar{s}$), not the dataset label distribution:
Even if the dataset had 100% “true” labels or a 50/50 mix, SE would be calculated from the model’s correct/incorrect outcomes.

**Case A: Bernoulli (accuracy-like)**
* Observed model accuracy $\bar{s} = 0.55$

$$
SE_{Bernoulli} = \sqrt{\frac{\bar{s}(1-\bar{s})}{n}} = \sqrt{\frac{0.55 \cdot 0.45}{200}} \approx 0.035
$$

$$
CI_{95\%} = 0.55 \pm 1.96 \cdot 0.035 \approx [0.481, 0.619]
$$

**Case B: Fractional metric (not Bernoulli)**
* Suppose per-item F1 scores with mean $\bar{s}=0.62$ and sample variance $s^2=0.04$.

$$
SE_{CLT} = \sqrt{\frac{s^2}{n}} = \sqrt{\frac{0.04}{200}} \approx 0.014
$$

$$
CI_{95\%} = 0.62 \pm 1.96 \cdot 0.014 \approx [0.592, 0.648]
$$

*  If one incorrectly treated F1 as Bernoulli with $p=0.62$:

$$
SE_{Bernoulli} = \sqrt{\frac{0.62 \cdot 0.38}{200}} \approx 0.034
$$

$$
CI_{95\%} = 0.62 \pm 1.96 \cdot 0.034 \approx [0.553, 0.687]
$$

The Bernoulli assumption is too conservative for fractional metrics like F1.

## Calculating Error Bars of Correlated (Clustered) Questions
If the dataset contains correlated clusters (e.g., the same question translated into multiple languages), independence is violated. 
Naive SE calculations yield confidence intervals that are too narrow, creating *overly optimistic* error bars. 
The correction is to compute clustered standard errors, which explicitly model correlation within groups.

## Paired Model Comparison
The naive comparison above misses an opportunity to reduce the standard error when two models evaluate the same set of questions.
When two models are scored on the same items, evaluating their paired differences reduces variance:
1. The variance in the unpaired case $Var(\hat{\mu}_{A-B, unpaired}) = \frac{Var(s_A) + Var(s_B)}{n}$
2. The variance in the pairs case  $Var(\hat{\mu}_{A-B, paired}) = \frac{Var(s_A) + Var(s_B) - 2\,Cov(s_A, s_B)}{n}$

Since models typically agree on which items are easy or hard, paired variance is smaller, yielding tighter confidence intervals by subtracting their covariance. 


## Power analysis
Power analysis determines how many evaluation examples are needed to measure differences between models with statistical confidence. The core idea is to control both the probability of detecting a real difference (power) and the risk of a false positive (significance level $\alpha$).

Formally, the required sample size $n$ depends on three quantities: the desired confidence level, the minimum effect size $\Delta$ one wants to detect, and the variance of the evaluation metric $\sigma^2$. For a two-sided test with significance level $\alpha$ and power $1-\beta$, the sample size can be approximated as

$$
n \geq \frac{(z_{1-\alpha/2} + z_{1-\beta})^2 \, \sigma^2}{\Delta^2}
$$

where $z_{p}$ is the $p$-quantile of the standard normal distribution.

For instance, if $\hat{\sigma}^2 = 0.04$ and $\Delta = 0.02$, achieving 80% power at $\alpha = 0.05$ requires thousands of samples, reflecting the difficulty of detecting small differences in metrics like precision. Without such calculations, evaluations risk being underpowered, leading to unstable or misleading comparisons between models.


## Interpreting Statistical Significance
Statistical significance does not always mean one model is truly better.
1. A model may appear “significantly better” on one benchmark yet perform worse elsewhere, or the gap may be too small to matter in practice.
2. P-values and confidence intervals only measure how unlikely the results would be *if the null hypothesis were true*. The null assumes the two models have the same average performance and any gap is random. Rejecting it means the difference is unlikely due to chance on that dataset, not that it generalizes broadly.
3. Over-reliance on significance can inflate leaderboards with narrow, dataset-specific gaps that don’t reflect real progress.

## Resource
[Paper](https://arxiv.org/pdf/2411.00640)