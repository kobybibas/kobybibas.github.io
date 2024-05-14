---
title: "[Summary] Semi-supervised learning made simple with self-supervised clustering" 
date: 2024-05-11
tags: 
    - Clustering
    - Self-supervision
    - Semi-supervision
    - Classification
    - Computer vision
draft: false 
---

## TL;DR
Take advantage of the few annotations available in the semi-supervised setting to learn even better representations.
Our main intuition is to replace the cluster centroids with class prototypes learned with supervision. In this way, unlabeled samples will be clustered around the class prototypes, guided by the self-supervised clustering-based objective.


## Method
Essentially the method trains a model by a joint optimization of a supervised loss on the labeled data and a self-supervised loss on the unlabeled data using the same loss function (cross-entropy).

**Unlabeled data.** Given two correlated views of an input image:
1. Feeding the images to an encoder (x,x^)
2. Use an additional bias-free linear layer to produce (h, h^)
3. Create a clustering assignment of h^ by applying the softmax function. This acts as the pseudo-label y^.
4. y^ is used as the target label in the cross-entropy loss for softmax(h). Note that a stop-gradient operation is performed, so that the gradient is not propagated through the pseudo-label.

Some notes:
* The length of h and h^ is equal the number of clusters. The number of clusters is pre-defined. 
* The cluster assignment is not needed: The difference between h and h^ can directly be used as the loss function. However, the clustering helps to prevent collapsing of the representation: where all samples are assigned to the same cluster. 
* Intuitively, the loss leverages cluster assignments as a proxy to minimize the distance between latent representations (h, ˆh). As a by-product, the objective also learns a set of cluster.

**Labeled data.**
This can be achieved by optimizing the same loss function in Eq. 2
while replacing the pseudo-label with the ground-truth la-
bel when available:


![Method](/posts/20240511_Semi-supervised learning made simple with self-supervised clustering/summary.md)

## Limitations
* In the paper's experiment, the authors considered the settings where all classes have some labaled samples. It's not clear how the method can be used where there are unknown clusters with no annotated samples. 
* 

## Resource
[Arxiv](https://arxiv.org/pdf/2306.07483): published in CVPR 2022

[GitHub](https://github.com/pietroastolfi/suave-daino?tab=readme-ov-file)
