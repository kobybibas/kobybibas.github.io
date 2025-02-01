---
title: "[Summary] ContraNorm: A Contrastive Learning Per-spective on Oversmoothing and beyond"
date: 2025-02-01
tags: 
- Transformer
draft: false 
---

## TL;DR 
Oversmoothing is a common phenomenon in Transformers, where performance worsens due dimensional collapse of representations, where representations lie in a narrow cone in the feature space.
The authors analyze the Contrastive lose and extracted the term that prevent this collapse. By taking a gradient descent step with respect to the feature later, they come up with the ContraNorm layer that leads to a more uniform distribution and prevents the dimensional collapse.

![Dimensional collapse](/posts/20250201_contranorm/dimensional_collapse.png)


## On dimension collapse in Transformers
Representation collapse in Transformers occurs when deeper layers cause features to lose expressivity, leading to performance decline. 
This phenomenon is usually measured using attention map similarity: as attention map similarity converges to one, feature similarity increases.

However, using the attention map similarity overlooks cases of **dimensional collapse**:  it fails to capture cases where representations span a low-dimensional space without fully collapsing.

The figure below shows the singular values distribution. It illustrates how the proposed method helps to distribute singular values more uniformly across dimensions.
![Singular values](/posts/20250201_contranorm/singular_values.png)


## Method
The ContraNorm is proposed to mitigate the dimensional collapse in Transformers by applying the following transformation:
![Singular values](/posts/20250201_contranorm/contranorm_layer.png)
* \(H_b\) is the feature matrix 
* \(\tau\) is the temperature scaling 
* \(s\) is an hyper-parameter corresponds to the gradient descent step 

**Deriving the ContraNorm layer**. Looking at the contrastive loss, it can be decoupled into alignment loss and uniformity loss
![contrastive loss](/posts/20250201_contranorm/CONTRASTIVE_LOSS.png)
1. The alignment loss (the numerator) encourages feature representations for positive pairs to be similar, thus being invariant to unnecessary noises. However, training with only the alignment loss will result in a trivial solution where all representations are identical.
2. The uniformity loss (the denominator) maximizes average distances between all samples, resulting in embeddings that are roughly uniformly distributed in the latent space. 

An alternative approach to address oversmoothing is to directly incorporate the uniformity loss into the training objective. However, experiments shows that this method has limited effectiveness
Instead, a normalization integrated into the model:
1. Taking the derivative of the uniformity loss
2. Apply a single step of gradient descent  
3. In addition, LayerNorm(*) is applied on top to guarantee a more uniform distribution of the features in each update step.

## Limitations
1. The method introduces three additional hyperparameters: (i) the uniformity strength, (ii) temperature scaling, and (iii) LayerNorm parameters. 
2. While ContraNorm helps alleviate dimensional collapse, there may be a trade-off between reducing collapse and maintaining feature diversity. Preventing the model to compress the input may slow down the model convergence.

## Resources
[arxiv](https://arxiv.org/pdf/2303.06562), published in ICLR 2023
