---
title: "[Summary] Bag of Tricks for Multimodal AutoML with Image, Text, and Tabular Data"
date: 2025-09-09
tags:
- AutoML
- Multimodal Learning
- Image Processing
- Text Analysis
- Tabular Data
draft: false
---

## TL;DR
Many machine learning systems deal with multimodal data, however there's no comprehensive study examining different design choices and implementation tricks across modalities. 
The paper surveys common "tricks" for multimodal systems and found the most consistently effective techniques are: (i) basic training tricks such as gradient clipping and learning-rate warmup, (ii) late fusion using pretrained unimodal encoders (especially CLIP-aligned image and text encoders), (iii) auxiliary cross-modal alignment objectives, (iv) simple input-level augmentation, and (v) modality dropout and learnable embeddings for handling missing inputs.

![multimodal example](/posts/20250909_bag_of_tricks_multimodal_automl/multimodal_example.png)

## Motivation
While multimodal learning has gained traction, practitioners face recurring design decisions without clear guidance on which approaches work best. 
Multimodal learning faces recurring design questions:
1. How should different modalities be fused? (early feature fusion vs. late decision-level fusion)
2. What form of data augmentation—input-level vs. feature-level—provides the strongest gains?
3. Does adding cross-modal alignment objectives help in supervised fine-tuning?
4. How can models remain robust when one or more modalities are missing at training or inference?

The study collects strategies scattered across prior work and evaluates them under one framework. 
The experimental setup fine-tunes three modality-specific backbones: Swin Transformer Large (images), DeBERTa-V3 (text), FT-Transformer (tabular).

## Basic tricks
Certain generic training adjustments consistently improved performance:
1. Greedy soup: averaging the top three checkpoints along a single training trajectory.
2. Gradient clipping: mitigates exploding gradients.
3. Learning-rate warmup: stabilizes optimization in early epochs.

The authors report little benefit from layerwise learning-rate decay, where distinct learning rates are set to each layer with shallower layers assigned higher learning rates and smaller rates for deeper layers.

## Multimodal Fusion Strategies
Fusion strategies fall into two categories:
1. Early fusion: concatenate features from all modalities before classification.
2. Late fusion: independent modality encoders, with predictions merged at the end.

The study found Late fusion to be more effective. Among variants:
a. LF-MLP: concatenated CLS token features fused by an MLP.
b. LF-Aligned (CLIP encoders): leveraging pretrained aligned unimodal encoders yields the strongest results, except on text+tabular tasks.

## Cross-modal Alignment
Cross-modal alignment projects data from different modalities into a shared embedding space.
Two alignment losses were compared:
1. Positive-only approach: minimizing the KL divergence losses among different modalities within each sample
2. Positive+negative approach: uses a loss (InfoNCE) which additionally separates modalities across different samples.
While the positive+negative approach shows significant performance improvement on the image+text combination, the positive-only is better than Baseline+ across all modality compositions. 

## Multimodal Data Augmentation
Data augmentation enriches datasets by creating variations of existing data.
Three approaches were evaluated:
1. Applying augmentation operations directly to the input data.
2. Augmenting samples in the feature space within the model. Two methods were considered:
   1. Interpolates latent vectors and labels to generate new samples.
   2. Tains an augmentation network to learn how to jointly augment all the modalities of each sample in the feature space.

Despite simplicity, input-level augmentation consistently outperforms the others.

## Handling Modality Missingness
In real-world applications, it is common for some samples to have incomplete modalities during either training or deployment.
Tested strategies include:
1. Modality Dropout: randomly masking modalities during training with a predefined probability.
2. Learnable Embeddings: encoding missing categorical/text/image fields as trainable vectors.
3. Traditional imputation: mean-fill for numeric, special-category for categorical, CLS token for empty text, zero tensor for missing images.
Learnable embeddings boost image+tabular and full three-modality settings, while dropout is strongest for text+tabular and all-modality mixes
.
### Integrating Bag of Tricks
The authors created a model pool trained with different tricks and build ensembles. 
A learnable weighted ensemble selection outperforms any individual strategy, highlighting that many tricks are complementary but not universally effective.

## Takeaways 
1. Greedy soup, gradient clipping, and learning-rate warmup are low-cost performance gains
2. Late fusion with pretrained aligned encoders dominates across fusion strategies
3. Cross-modal alignment objectives improve fine-tuning, with KL divergence giving broader benefits than InfoNCE
4. Simple input-level augmentation remains best
5. Modality dropout and learnable embeddings both enhance robustness to missing inputs
6. Ensembles of models using different tricks outperform any single approach

## Resource
[Paper](https://arxiv.org/abs/2412.16243)