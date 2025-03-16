---
title: "[Summary] Towards Monosemanticity: Decomposing Language Models With Dictionary Learning"
date: 2025-03-15
tags: 
- Mechanistic interpretability
- Sparse autoencoder
draft: false 
---

## TL;DR 
Transformer models often map multiple concepts to the same neuron, making it unclear what features they learn. This work makes inner representations interpretable by using a sparse autoencoder layer to map neurons to concepts. This method was found to extract relatively monosemantic concepts, can be used to steer transformer generation, be able to understand that  512 neurons can represent tens of thousands of features.

## Method
A major challenge in reverse engineering neural networks is the curse of dimensionality. As models grow, the latent space volume increases exponentially. 

We use a sparse autoencoder, a weak dictionary learning algorithm, to generate interpretable features from a trained model. This method decomposes MLP activations into more interpretable features. We decompose into more features than neurons, assuming the MLP layer uses superposition. We demonstrate this with a one-layer transformer with a 512-neuron MLP layer, training sparse autoencoders on activations from 8 billion data points. We analyze 4,096 features learned in one run.

![teaser](/posts/20250315_towards_monosemanticity/setup.png)

If linear directions are interpretable, there should be a basic set of meaningful directions (features) to decompose models into.

#### What makes a good decomposition?
A useful decomposition approximates MLP activations as a sparse weighted sum of features if:
1. We can interpret when each feature is active.
2. We can interpret the downstream effects of each feature.
3. The features explain a significant portion of the MLP layer's functionality (measured by loss).

Sparse autoencoder shows a more psormising direction if interperate the inner represnation than arhictature change: 
several attempts were made to induce activation sparsity during training to produce models without superposition that fails to result in cleanly-interpretable neurons.


## Validating the appraoch

Thus we ended up using a combination of several additional metrics to guide our investigations:
1. Manual inspection: Do the features seem interpretable?
2. Feature density: we found that the number of “live” features and the percentage of tokens on which they fire to be an extremely useful guide. (See appendix for details.)
3. Reconstruction loss: How well does the autoencoder reconstruct the MLP activations? Our goal is ultimately to explain the function of the MLP layer, so the MSE loss should be low.


![teaser](/posts/20250315_towards_monosemanticity/concept_activation.png)


## Limitations
1. This study uses a one-layer transformer. It's unclear if the results generalize to larger models like GPT.
2. There's no quantitavie way to say if the autoencoder apporach works
3. We finally note that the features in this section are cherry-picked to be easier to analyze. Defining simple computational proxies for most features we find, such as text concerning fantasy games, would be difficult, and we analyze them in other ways in the following section.

## Resources
[Source](https://transformer-circuits.pub/2023/monosemantic-features)
