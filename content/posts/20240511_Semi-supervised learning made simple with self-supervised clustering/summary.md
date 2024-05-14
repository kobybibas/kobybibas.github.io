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
In self-supervised learning there are no guarantees that representations will organize the clusters according to their semantic classes.
In many cases, labels are partially available and can be leveraged to improve the semantic clustering of self-supervised learners.  
In this paper, for the labeled samples the authors propose to replace the cluster centroids with class prototypes learned with supervision. In this way, unlabeled samples will be clustered around the class prototypes, guided by the self-supervised clustering-based objective.


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

**Leveraging labeled data.**
By learning with unlabeled data only there are no guarantees that representations will organize the clusters according to the class labels. 
With annotated data, the pseudo-label can be replaced with the ground-truth label when available:
![equation](/posts/20240511_Semi-supervised learning made simple with self-supervised clustering/semi_supervised_learning_eq.png)
where y is the ground truth label, y^ is the pseudo label, and $l$ is the loss function.

Since the authors balance the labeled and unlabeled loss by 1:1, implementation wise they concatenate the labeled and unlabeled samples, the pseudo label and true label. Then they feed them to the same cross entropy loss.

![Method](/posts/20240511_Semi-supervised learning made simple with self-supervised clustering/method.png)

## Limitations
* It's not clear it the method works when there are some classes with no label data. Potentially, this classes can be spread across clusters with not meaningful semantic separation. 
* There is a hidden hyperparameter of balancing the labeled and unlabeled loss. In the paper, the authors assume it's 1 without explicitly mention and discuss it. The method is probably sensitive for this hyperparameter
  

## Resource
[Arxiv](https://arxiv.org/pdf/2306.07483): published in CVPR 2022

[GitHub](https://github.com/pietroastolfi/suave-daino?tab=readme-ov-file)
