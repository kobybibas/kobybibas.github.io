---
title: "[Concept] The Attention Mechanism of Transformer Models"
date: 2025-08-22
tags:
- Transformers
- Attention
draft: false
---

*  query vector $q\in\mathbb{R}^d$
*  keys $K\in\mathbb{R}^{n\times d}$ (rows $k_i^\top$)
*  values $V\in\mathbb{R}^{n\times d_v}$

Steps to compute the attention
1.  Compute similarities \(s_i=\frac{\langle q,k_i\rangle}{\sqrt{d}}\)
2.  Convert scores to a probability like values: \(p_i=\mathrm{softmax}(s)_i=\frac{e^{s_i}}{\sum_j e^{s_j}}\)
3. Aggregate: \(\mathrm{Attn}(q,K,V)=\sum_{i=1}^n p_i\, v_i\)

In the vanilla Transformer setup:
* $q = xW^Q$,
* $k = xW^K$,
* $v = xW^V$.

Query, key, and value all come from the same token embedding \(x\), but with different learned linear projections:
They start from the same information, but the model is free to learn different subspaces for “asking” (queries), “addressing” (keys), and “answering” (values).

**Why $\tfrac{1}{\sqrt d}$ scaling?**
If \(q,k_i\sim\mathcal N(0,I)\), then \(\langle q,k_i\rangle\) has variance \(d\). 
Dividing by \(\sqrt d\) keeps the logits’ variance $\approx 1$, avoiding softmax saturation and stabilizing gradients.


![Attention](content/posts/20250822_attention_mechanism_of_transformer/attention.png) 
*Taken from https://learnopencv.com/attention-mechanism-in-transformer-neural-networks/*

## Probabilistic/kernel view
Let \(\kappa(q,k)=\exp\!\big(\tfrac{\langle q,k\rangle}{\sqrt d}\big)\). Then

$$
\mathrm{Attn}(q,K,V)
= \frac{\sum_i \kappa(q,k_i)\, v_i}{\sum_j \kappa(q,k_j)}
$$

This is the *Nadaraya–Watson kernel regression* estimator with exponential (dot-product) kernel. 
* Keys are inputs
* Values are labels 
* The attention predicts a kernel-weighted average label at query \(q\).




## Batched/matrix form + multi-head

For a batch of queries $Q\in\mathbb{R}^{m\times d}$:

$$
\mathrm{Attn}(Q,K,V)=\mathrm{softmax}\!\Big(\frac{QK^\top}{\sqrt d}\Big) V
\quad\text{(row-wise softmax).}
$$

**Multi-head:** learn $h$ separate linear maps
$QW_i^Q,\ KW_i^K,\ VW_i^V$ (lower-dim $d_h=d/h$), compute $h$ attentions in parallel, then concatenate and mix with $W^O$.
Intuition: each head is a different kernel / subspace, giving **disentangled relational channels** and richer low-rank structure.

**Causal/mask:** add a mask $M$ with $-\infty$ on disallowed positions before the softmax:

$$
\mathrm{softmax}\!\Big(\frac{QK^\top}{\sqrt d}+M\Big)V.
$$

**Shapes & cost:** For seq length $n$, width $d$, cost is $O(n^2 d)$ time and $O(n^2)$ memory (the attention matrix). This motivates long-context variants (sparse, linearized, low-rank, etc.).

## Gradients & stability (what really matters in practice)

Let $A=\mathrm{softmax}(QK^\top/\sqrt d)$. Then output $Y=AV$.

* **Backprop wrt V:** $\partial L/\partial V = A^\top (\partial L/\partial Y)$. So values get gradients redistributed by attention weights.
* **Backprop wrt scores $S=QK^\top/\sqrt d$:** uses Jacobian of softmax; roughly, it pushes scores to increase probability mass on value directions that reduce loss. Saturation (very peaky $A$) kills gradients → hence scaling, normalization, and temperature tuning matter.
* **Init/norm:** Pre-LN Transformers and RMSNorm/LayerNorm stabilize the score distribution; residual connections keep signal/gradient highways.
* **Temperature:** dividing logits by $T>1$ (equivalently multiplying by $<1$) flattens attention; $T<1$ sharpens.

## Deeper lenses (quick hits)

* **Low-rank view:** $QK^\top$ is at most rank $d$. Multi-head attention composes multiple low-rank bilinear forms → expressive yet parameter-efficient.
* **Message passing:** With tokens as nodes, attention does content-based weighted aggregation—like a fully connected GNN layer with learned edge weights.
* **Energy-based view:** Softmax attention arises from a linear energy plus entropy regularizer; alternatives (sparsemax, entmax) alter the regularizer → sparse supports.
* **Linear attention:** Replace softmax kernel by a feature map $\phi(\cdot)$ with $\kappa(q,k)\approx \phi(q)^\top\phi(k)$ to get $(\phi(Q)\,(\phi(K)^\top V))$ and **O(n d^2)** time, **O(n d)** memory.
* **KV caching (inference):** Reuse past $K,V$ to serve next query in $O(n d)$ per step.
