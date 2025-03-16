---
title: "[Summary] Towards Monosemanticity: Decomposing Language Models With Dictionary Learning"
date: 2025-03-15
tags: 
- Mechanistic interpretability
- Sparse autoencoder
- Transformer
- Dictionary learning
draft: false 
---

## TL;DR 
Transformer models often map multiple concepts to the same neuron, making it unclear what features they learn. This work makes inner representations interpretable by using a sparse autoencoder layer to map neurons to concepts. 
This method extracts relatively monosemantic concepts, can steer transformer generation, and shows that 512 neurons can represent tens of thousands of features.

## Method
A major challenge in reverse engineering neural networks is the curse of dimensionality: as models grow, the latent space volume increases exponentially.

A sparse autoencoder, a weak dictionary learning algorithm, is used to generate interpretable features from a trained model. If linear directions are interpretable, there should be a basic set of meaningful directions (features) to decompose models into. 
This method decomposes multi-layer perceptron (MLP) activations into more interpretable concepts. Demonstrated with a one-layer transformer with a 512-neuron MLP layer, sparse autoencoders are trained on activations from 8 billion data points. Analysis focuses on 4,096 features.

![teaser](/posts/20250315_towards_monosemanticity/setup.png)

#### What makes a good decomposition?
A useful decomposition approximates MLP activations as a sparse weighted sum of features if:
1. The conditions under which each feature is active can be interpreted.
2. The downstream effects of each feature can be interpreted.
3. The features explain a significant portion of the MLP layer's functionality (measured by loss).

Sparse autoencoders show a more promising direction for interpreting inner representations than architectural changes. 
Attempts to induce activation sparsity during training to produce models without superposition failed to result in cleanly interpretable neurons.

## Validating the approach
Several metrics guide the investigation of the validity of sparse autoencoders to represent internal concepts:
1. Manual inspection: Do the features seem interpretable?
2. Feature density: The number of “live” features and the percentage of tokens on which they fire.
3. Reconstruction loss: The goal is to explain the function of the MLP layer, so the MSE loss should be low.

![teaser](/posts/20250315_towards_monosemanticity/concept_activation.png)

## Limitations
1. This study uses a one-layer transformer. It's unclear if the results generalize to larger models like GPT.
2. There's no quantitative way to confirm if the autoencoder approach works.
3. Features were picked for easier analysis. It's not clear how generalizable the results are to more vague features like "fantasy games".

## Resources
[Source](https://transformer-circuits.pub/2023/monosemantic-features)
