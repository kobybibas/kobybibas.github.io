---
title: "[Summary] Estimating a Treatment Effect with Causal Inference"
date: 2026-09-28
tags:
  - Causal Inference
  - Treatment Effect
  - Conditional Average Treatment Effect
  - S-Learner
  - T-Learner
draft: false
---

## Intro

Machine learning models predict by the correlation between features and labels. For an outcome under a different treatment, that correlation can give the wrong answer.

Consider the following example. We have three samples: one got treatment \(T=0\), and two got treatment \(T=1\).

| sample | treatment \(T\) | Observed outcome \(Y\) | \(Y(0)\), had they received \(T=0\) | \(Y(1)\), had they received \(T=1\) |
| ------ | ------------- | -------------------- | ------------------------------- | ------------------------------- |
| 1      | 0             | 9                    | 9                               | 12                              |
| 2      | 1             | 3                    | 0                               | 3                               |
| 3      | 1             | 3                    | 0                               | 3                               |

A naive way to measure whether treatment \(T=1\) or \(T=0\) is more helpful:

$$
\text{Observed gain} = \frac{3+3}{2} - 9 = 3 - 9 = -6.
$$

However, let's say we have an oracle that provides, for each sample, what the outcome would have been given a different treatment. If we consider the causal effect, the true gain is

$$
\operatorname{mean}(Y(1)) - \operatorname{mean}(Y(0)) = 6 - 3 = 3.
$$

We get the complete opposite conclusion!

## Measurement of Interest in Causal Learning

A common quantity in causal learning is the average treatment effect (ATE):
$$
\tau = \mathbb{E}[Y(1) - Y(0)]
$$
The issue is the effect can depend on the sample we treat. **Heterogeneous treatment effect (HTE)**, or conditional average treatment effect (CATE), is the effect given the features \(x\). 

Under three common assumptions in causal inference:
1. Consistency: If a sample received treatment \(t\), the outcome we recorded is \(Y(t)\). Also, one sample's treatment does not change another's.
2. No unmeasured confounders: Given \(x\), treatment does not depend on the potential outcomes, i.e., every confounder is inside \(x\).
3. Common support: For the features we see, all possible treatments have a nonzero chance.  
The CATE can be identified from the observed data:
$$
\tau(x) = \mathbb{E}[Y \mid x, T=1] - \mathbb{E}[Y \mid x, T=0]
$$

### When to Use Causal Learning VS the Typical Correlation Based Learning

The question we care about decides whether we need a causal effect at all:

| Question                                           | Description                                                    | Approach                                                    |
| -------------------------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------- |
| We want \(Y\) under the current policy               | We observe the outcome we want to predict.                     | Supervised learning.                                        |
| We want the gain of the treatment we already give  | For the treated samples we never see the counterfactual \(Y(0)\). | The average treatment effect (ATE), on the treated samples. |
| We want who benefits if we change who gets treated | For each sample we see only one of \(Y(0)\) and \(Y(1)\).          | The conditional effect (CATE).                              |

When we need the causal effect, there are different tools we can use.

| Situation                                             | What we do                                                                                  |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| We can randomize                                      | Run the A/B test. Fit a CATE model if we need who benefits.                                 |
| We cannot randomize, but the confounders are measured | Estimate the effect from observational data: S-learner or T-learner. |

### Machine Learning Approach for Counterfactual Inference

**S-learner.** In this approach we train a model with the treatment as one of the model features

$$
\mu(x, t) := \mathbb{E}[Y \mid x, T=t]
$$

and the estimated CATE is

$$
\hat{\tau}(x) = \hat\mu(x, 1) - \hat\mu(x, 0).
$$

**T-learner.** The T-learner fits a separate model for each treatment:

$$
\mu_t(x) = \mathbb{E}[Y \mid x, T=t]
$$

Under the assumptions above, the estimated CATE is

$$
\hat{\tau}(x) = \hat\mu_1(x) - \hat\mu_0(x)
$$

### Model Complexity

It's tempting to use a linear estimator to model the causal effect. However, if the true model is not linear, we get a biased estimator which results in a wrong conclusion. This means we must follow the true model, which is not likely in the real world. So we want to use a more powerful model class (random forest, NN).

**Proof of the statement above.** Let \(x \in \{-1, 1\}\), each with probability \(1/2\), and let \(T\) be independent of \(x\). The true outcome is

$$
Y(t) = c \cdot x \cdot t.
$$

The effect is the difference of the two potential outcomes:

$$
\tau(x) = Y(1) - Y(0) = c \cdot x - 0 = c x.
$$

So at \(x=1\) the effect is \(c\), and at \(x=-1\) it is \(-c\).

A linear model gives the effect one number \(\gamma\) for every \(x\):

$$
Y(t) = \beta x + \gamma \cdot t.
$$

One number cannot equal both \(c\) and \(-c\). The closest single number is their average. Since each value of \(x\) has probability \(1/2\),

$$
\gamma = \frac{1}{2} \cdot c + \frac{1}{2} \cdot (-c) = 0.
$$

The error at \(x=1\) is then

$$
|\tau(1) - \gamma| = |c - 0| = |c|.
$$

We can pick \(c\) as large as we want, so the error is arbitrarily large. While the average effect is right, the effect for a given \(x\) is not.

#### Glossary

**Treatment** \(T \in \{0,1\}\). The intervention we set. A value of \(T\) is written \(t\), and the treatment of sample \(i\) is \(T_i\).

**Covariates** \(x\). Also called features. Background variables that can affect the outcome.

**Potential outcomes** \(Y(1)\) and \(Y(0)\). The outcome the same sample would have under \(T=1\) and under \(T=0\). 

**Counterfactual**. The potential outcome we do not observe. If the sample got \(T=1\), the counterfactual is \(Y(0)\).

**CATE** \(\tau(x) = \mathbb{E}[Y(1) - Y(0) \mid x]\). The effect of treatment for a sample with features \(x\). Heterogeneous treatment effect (HTE) is the same quantity: the effect is not one number for everyone.

**ATE** \(\mathbb{E}[\tau(x)]\). The average of the CATE over samples. A randomized difference of means.

**Outcome model** \(\mu(x, t) = \mathbb{E}[Y \mid x, T=t]\). A supervised model of the outcome. The S-learner fits one \(\mu\). The T-learner fits \(\mu_1(x)\) and \(\mu_0(x)\) separately.

**Confounding**. A third factor that affects both \(T\) and \(Y\), so the observed contrast is not the effect of setting \(T\).

**Covariate adjustment**. Estimate \(\tau(x)\) from \(\mathbb{E}[Y \mid x, T=1] - \mathbb{E}[Y \mid x, T=0]\). The S-learner and the T-learner do this with \(\mu\).

#### Reference

<https://causaldm.github.io/Causal-Decision-Making/0_Motivating_Examples/CEL.html>

<https://www.youtube.com/watch?v=gRkUhg9Wb-I>  

<https://www.youtube.com/watch?v=g5v-NvNoJQQ>
