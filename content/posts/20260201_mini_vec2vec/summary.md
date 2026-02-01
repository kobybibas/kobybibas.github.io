---
title: "[Summary] Mini-vec2vec: Scaling Universal Geometry Alignment with Linear Transformation"
date: 2026-02-01
tags:
  - Embedding Alignment
  - Unsupervised Learning
draft: false
---

## TL;DR

Mini-vec2vec is an unsupervised framework for aligning different embedding spaces without paired data. It transforms vectors between models (e.g., BERT to T5) using a linear assumption and iterative refinement. This approach achieves alignment quality comparable to the popular CycleGAN-based vec2vec method while reducing computational overhead.

## Motivation

Deep learning models often converge toward a geometric structure representing the underlying data manifold. However, direct comparison across models is usually impossible without expensive supervised re-training or ground-truth pairs. Existing unsupervised methods like vec2vec utilize complex generative adversarial setups (CycleGAN) that are slow and difficult to tune. The authors propose that a simpler linear transformation, combined with robust anchor-based matching, can recover these alignments more efficiently.

## Problem Statement

We assume access to two pools of text embeddings, each from a different encoder. Our task is to align the encoders' embedding spaces without paired examples.

The solution should learn a function \(f\) such that:

$$f(E_A(z)) \approx E_B(z)$$

where \(z\) represents a datapoint \(z \in Z\).

## Mini-vec2vec Pipeline

The mini-vec2vec pipeline operates in three distinct phases designed to find correspondences in unaligned sets \(E_A\) and \(E_B\).

### A. Pre-processing

Each embedding set undergoes mean-centering and normalization to the unit hypersphere.

### B. Anchor-based Matching

The core innovation involves creating a "relative representation" of vectors.

1. **Clustering:** Independent k-means runs identify centroids in both spaces:
$$\{c_j^A\}_{j=1}^k, \quad \{c_j^B\}_{j=1}^k$$

2. **Similarity Matrices:** Computing intra-space similarity matrices for these centroids (similarity matrices between the centroids of the same embedding space):
$$S_{ij}^A = \cos(c_i^A, c_j^A), \quad S_{ij}^B = \cos(c_i^B, c_j^B)$$

3. **Quadratic Assignment Problem (QAP):** Solving for an optimal permutation that aligns centroids from \(E_A\) to \(E_B\) based on their relational structure, i.e., finding the optimal assignment to maximize correlation of the similarity metrics. Denote \(\pi: \{1, 2, \ldots, k\} \rightarrow \{1, 2, \ldots, k\}\) as the permutation from the set \(\Pi_k\):

$$\pi^* = \arg \max_{\pi \in \Pi_k} \sum_{i=1}^k \sum_{j=1}^k S_{ij}^A S_{\pi(i) \pi(j)}^B$$

4. **Relational Encoding:** Each vector is redefined by its cosine similarity to these aligned anchors. For example, for a vector in space $A$:
$$\mathbf{r}_i^A = [\cos(\mathbf{x}_i^A, \mathbf{c}_1^A), \cos(\mathbf{x}_i^A, \mathbf{c}_2^A), \ldots, \cos(\mathbf{x}_i^A, \mathbf{c}_k^A)]$$

### C. Mapping and Refinement

Using the relational features, pseudo-parallel pairs can then be identified. For each vector in space \(A\), we find its \(k\) nearest neighbors in space \(B\) based on their relational representations, then pair it with the **average** of those neighbors:

$$\text{Pair}_i = \left(\mathbf{x}_i^A, \frac{1}{k} \sum_{j \in \text{top-}k} \mathbf{x}_j^B\right)$$

The neighborhood lookup is done in the relative space (which aligns the two spaces), but the actual pairing happens in absolute space \(B\).

The optimal linear transformation is then computed via Procrustes analysis using SVD:

$$\mathbf{W}^* = \mathbf{VU}^T \quad \text{where} \quad \mathbf{U}\Sigma\mathbf{V}^T = \text{SVD}(\mathbf{A}^T\mathbf{B})$$

Here, \(\mathbf{A}\) and \(\mathbf{B}\) are matrices containing all the pseudo-parallel pairs.

This process repeats iteratively, refining the transformation and finding increasingly accurate pairs in each round.

## Limitations
1. mini-vec2vec assumes the correctness of the Platonic Representation Hypothesis: if two models have captured fundamentally different aspects of the data (e.g., a code-only model vs. a prose-only model), the anchor-based matching will likely fail
2. The linear mapping might struggle with models trained on vastly different tokenization schemes where the manifold topology diverges
3. The QAP step might take significant time for large \(k\).

## Resources
[GitHub Repository](https://github.com/guy-dar/mini-vec2vec)

[arXiv Paper](https://arxiv.org/pdf/2510.02348)

[Original vec2vec Paper](https://arxiv.org/pdf/2505.12540)
