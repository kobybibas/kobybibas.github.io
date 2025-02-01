---
title: "[Summary] ContraNorm: A Contrastive Learning Per-spective on Oversmoothing and beyond"
date: 2025-02-01
tags: 
- Transformer
draft: false 
---

## TL;DR 
Oversmoothing is a common phenomenon in Transformers, where performance worsens due to representations converge to dimensional collapse, where representations lie in a narrow cone.
The authors analyze the Constructive lose and extracted the term that prevent this collapse. By taking a gradient descent step with respect to the feature later, they come up with the ContraNorm layer that implicitly shatters representations in the embedding space, leading to a more uniform distribution and a slighter dimensional collapse.

![Dimensional collapse](/posts/20250201_contranorm/dimensional_collapse.png)


## On dimension collapse in Transformers
Representation collapse is a known phenomenon in Transformers where as deep the Transformers becomes, features loss their expressivity power and the model decreased performance.
This phenomenon is usually evaluated using attention map similarity: as attention map similarity converges to one, feature similarity increases.

However, using the attention map similarity overlooks cases of **dimensional collapse**: representations in the latent embedding space might not shrink to one single point but span a low dimensional space.  In such cases, feature similarity may be relatively low, but representations still lose expressive power. 

The figure below shows the singular values distribution and shows it concentrated around 40 dimensions, while with the suggested method it's more uniform and span across 80 dimensions :
![Singular values](/posts/20250201_contranorm/dimensional_collapse.png)


## Method
The loss can be decoupled into alignment loss and uniformity loss

1. The alignment loss encourages feature representations for positive pairs to be similar, thus being
invariant to unnecessary noises. However, training with only the alignment loss will result in a trivial solution where all representations are identical. \
2. Thanks to the property of uniformity loss, dimensional collapse can be solved effectively. Reviewing the form of uniformity loss in Eq. (2), it maximizes average distances between all samples, resulting in embeddings that are roughly uniformly distributed in the latent space, and thus more information is preserved. 

An alternative approach to address oversmoothing is to directly incorporate the uniformity loss into the training objective. However, our experiments reveal that this method has limited effectiveness
Instead, we propose a normalization layer that can be easily integrated into various models


Using the uniformity loss, 
1. take the derivative on the node representation 
2. take a single step of gradient descent  
After some algebra: the operation corresponds to the stop-gradient technique

Applyon later norm after the uniformiy loss update
LayerNorm(*)

![Singular values](/posts/20250201_contranorm/contranorm_layer.png)
Where Hb is the feature matrix, \tau is the temperature scaling (coming from unifomity loss) and s is an hyper-paramteres coressponds to the gardient descent step 

## Limitations

## Resources
[arxiv](https://arxiv.org/pdf/2303.06562), published in ICLR 2023
