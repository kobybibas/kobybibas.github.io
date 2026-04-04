---
title: "[Summary] A Decoder-Only Foundation Model for Time-Series Forecasting"
date: 2026-04-04
tags:
  - Time Series
  - Foundation Models
  - Forecasting
  - Transformers
draft: false
---

**TL;DR** TimesFM is a 200M-parameter decoder-only transformer trained on ~100B timepoints. It treats time-series patches the way LLMs treat tokens. In zero-shot, it matches or beats supervised SOTA on standard benchmarks while costing a fraction of LLM-based approaches like LLMTime.

## Motivation

Classical methods (ARIMA, ETS) fit per-series and cannot transfer across datasets. [LLMTime](https://arxiv.org/abs/2310.07820) repurposes GPT-3/LLaMA-2 as zero-shot forecasters but is expensive and underperforms supervised models.

NLP and CV have foundation models, but time series is harder: no discrete vocabulary, variable context/horizon/granularity, and far less public data. Can a dedicated time series foundation model match supervised SOTA zero-shot at a fraction of the cost of repurposing LLMs?

## Method

Given a context window of L timepoints, learn a model that predicts the next H steps, optimized with MAE.

The architecture is a decoder-only transformer that operates on patches instead of individual timepoints:
1. The input is split into non-overlapping patches of length p=32 (the time series analogue of a token)
2. Each processed by a residual block + positional encoding
3. Then fed through 20 causal self-attention layers (16 heads, dim=1280)
4. An output residual block maps each token to a prediction of length h=128

![TimesFM Architecture](/posts/20260404_timesfm_decoder_only_time_series/architecture.png)

Key design choices:
- Output patches are longer than input patches (h=128 vs. p=32), so forecasting 512 steps takes 4 auto-regressive steps instead of 16. Fewer steps means less error accumulation.
- Patch masking during training: a random number of timepoints (0 to p-1) are masked from the start of the first patch, so the model learns to handle any context length.

Training data: Google Trends, Wikipedia pageviews, other real-world sources, and synthetic data (ARMA, sinusoids, piecewise linear, step functions). ~100B timepoints total, mixed 80% real and 20% synthetic. The synthetic data fills granularity gaps, especially for sub-hourly frequencies underrepresented in real datasets. Loss: MSE. Trained for 1.5M iterations on 16 TPUv5e cores (~2 days for the 200M model).

![TimesFM Training Data](/posts/20260404_timesfm_decoder_only_time_series/training.png)

## Limitations

1. Point predictions only. The model outputs a single value per timestep, not a distribution. A probabilistic loss (e.g. quantile regression) would let you estimate confidence intervals.
2. No covariate support. The model takes only the raw time series as input. It cannot condition on external variables (e.g. temperature when forecasting electricity demand).
3. Interpretability. Like any deep model, TimesFM is a black box compared to statistical methods like ARIMA or ETS where you can inspect the learned coefficients directly.

## References

- [Paper (arXiv)](https://arxiv.org/abs/2310.10688)
- [LLMTime (Gruver et al., 2023)](https://arxiv.org/abs/2310.07820)
