---
title: "[Summary] Tabular Foundation Models: From TabPFN to TabFM"
date: 2026-07-12
tags:
  - Tabular Data
  - Foundation Models
  - In-Context Learning
  - Transformers
draft: false
---

## TL;DR

Decision tree models have dominated machine learning for tabular data, and previous attempts to apply transformers to tabular data ended with no success. Recently, a new paradigm has emerged: train a large transformer model on synthetic datasets, then apply in-context learning to a new dataset to predict the entire test set labels. Multiple recent papers (TabPFN, TabICL, TabFM) show this approach outperforms tuned decision-tree models. However, per-prediction inference is actually slower than trees.

## Why This Matters

Tabular data is the most common data type in real-world machine learning. While transformer models have taken over the computer vision and NLP domains, for tabular data the most used models are still the classic decision-tree ones: XGBoost, CatBoost, random forest. For roughly two decades, deep learning attempts to dethrone them failed - tables are heterogeneous (mixed types, different scales, missing values, irrelevant features), and every table stands for different data, so there was no obvious way to transfer knowledge between datasets.

## The Core Idea: Pretrain on Synthetic Data, Predict In-context

The core idea is to train a large transformer model on synthetic data, and perform in-context learning at the inference stage.

**Synthetic pretraining.** A large transformer is pre-trained to solve artificially generated prediction tasks sampled from a tabular dataset prior. The prior was designed based on Bayesian Neural Networks (BNNs) and Structural Causal Models (SCMs), to model complex feature dependencies and potential causal mechanisms underlying tabular data.

- **BNN prior** - one dataset = one random neural net: sample an NN architecture and its weights, then for each data point sample an input x, feed it through the BNN with sampled noise, and use the output y as the target.
- **SCM prior** - one dataset = one random causal graph: sample high-level parameters (dataset size, number of features), randomly create an acyclic graph that describes the causal structure of the features and the target, and sample data from that distribution. Because features and target are just nodes in the graph, this generates cases the BNN prior cannot: the target causing features, and hidden confounders.

![Graphs generating data in the TabPFN prior: a BNN, an SCM, and SCMs sampled from the prior](/posts/20260712_tabular_foundation_models/tabpfn_prior_graphs.png)

**In-context learning.** For a new tabular dataset, the model doesn't train on it. Instead, we feed the entire table into the context, and the transformer predicts the entire test set labels in one pass. There's no gradient updates and no hyperparameter tuning.

## The Models

**TabPFN v1 (2022)** - the proof of concept paper. Handles up to ~1K training samples, 100 features, 10 classes, classification only. It outperformed tuned boosted trees at that scale, ~230x faster - when counting the trees' training time.

**TabPFN v2 (Nature, 2025)** - the scaled-up version: alternating row and column-attention over the table, support for regression, categorical features, and missing values, up to 10K samples and 500 features. The headline number: in 2.8 seconds it outperforms an ensemble of the strongest baselines tuned for 4 hours.

**TabICL (ICML 2025)** - builds upon TabPFN to make it more scalable. TabPFN alternates row- and column-attention over the full n x m cell grid at every layer - it never compresses, so the expensive sample-wise attention costs O(n^2 m). TabICL instead compresses each row into a single fixed-size vector (column-then-row attention) and pays the O(n^2) attention once, over compressed rows, giving O(m^2 n + n^2) total. Dropping the m factor from the quadratic term is the entire scaling story: 500K rows, 3-10x faster on large tables.

**TabFM (Google, 2026)** - a hybrid of the two: TabPFN-style alternating row/column attention first, then TabICL-style row compression before the ICL stage, scaled up (32 estimators by default vs 8 in other tabular foundation models, larger embeddings, more ICL blocks) and pre-trained on hundreds of millions of synthetic SCM datasets. It currently tops the TabArena benchmark. Note there is no paper - only a blog post, inference code, and model weights.

## Limitations

- **Inference is costly.** If we need to perform one-by-one prediction, like in online inference, the reported speedup doesn't apply - you ship the training data with every prediction, and a forward pass through a large transformer is slower than a tree traversal. Context caching helps for batch settings, but trees remain far cheaper to serve.
- **Dataset scale is limited** relative to decision-tree models, both in rows and features (TabPFN v2: 10K rows; TabFM: 500 features, 10 classes; benchmarks top out at ~150K samples). Trees train comfortably on millions of rows.
- **The prior is the inductive bias.** These models approximate Bayesian inference under a synthetic prior. If your data comes from mechanisms far outside that prior's support, predictions degrade - and there is no training step to fix it.

## References

- TabPFN v1 - [A Transformer That Solves Small Tabular Classification Problems in a Second](https://arxiv.org/abs/2207.01848) (ICLR 2023)
- TabPFN v2 - [Accurate predictions on small data with a tabular foundation model](https://www.nature.com/articles/s41586-024-08328-6) (Nature, 2025)
- TabICL - [A Tabular Foundation Model for In-Context Learning on Large Data](https://arxiv.org/abs/2502.05564) (ICML 2025)
- TabFM - [Google Research announcement](https://research.google/blog/introducing-tabfm-a-zero-shot-foundation-model-for-tabular-data/) (2026)
- PFN framework - [Transformers Can Do Bayesian Inference](https://arxiv.org/abs/2112.10510) (ICLR 2022)
- Christoph Molnar - [The rise of tabular foundation models](https://mindfulmodeler.substack.com/p/tabular-ml-is-about-to-get-weird), [TabFM minus the hype](https://mindfulmodeler.substack.com/p/tabfm-minus-the-hype)
