---
title: "[Summary] On the Biology of a Large Language Model"
date: 2025-04-12
tags:
- Interpretability
- Transformers
- Attribution Graphs
- Large Language Model
draft: false
---

## TL;DR  
Large Language Models (LLMs) are often perceived as “black boxes,” making their decision-making and reasoning processes difficult to interpret. 
A novel method simplifies these complex models by replacing internal nonlinear layers with linear modules tailored to clearly understandable features. 
This approach reveals structured reasoning, planning behaviors, and even hidden intentions within the model’s computations.

## Method
Interpreting LLMs is challenging because individual neurons often represent multiple, unrelated concepts simultaneously (polysemanticity). 
To address this, the approach creates a simplified "replacement model", preserving most of the original model’s performance while enhancing interpretability through these steps:

1. **Fix Attention Layers:**
Keep the transformer’s attention mechanisms constant during analysis to isolate the complexity arising specifically from nonlinear layers (MLPs).

2. **Linear Approximation of MLP Layers:**
Each nonlinear MLP layer is replaced by a simpler linear layer trained to mimic the original MLP’s behavior using easily interpretable features. These simplified linear layers are called Cross-Layer Transcoders (CLTs). Specifically, this involves:
* Recording the inputs and outputs of each original MLP during a forward pass
* CLTs to replicate the MLP’s mappings via linear combination of pre-defined interpretable features
* Optimizing via standard regression techniques (e.g., minimizing mean squared error)
    
![replacement_model](/posts/20500412_biology_large_language_model/replacement_model.png)

3. **Introduction of Error Nodes:**
Since linear approximations alone cannot capture all complexity, additional “error nodes” are added to represent any residual information missed by the linear layers. This enhances the model’s fidelity to the original behavior.

4. **Creation of Attribution Graphs:**
The simplified model’s computations are visualized as attribution graphs, which clearly display how different features influence each other. In these graphs:
* Nodes represent input tokens, model outputs, or active internal features.
* Edges quantify the strength of influence between features.
* Paths with negligible contributions are pruned, and related features are combined into “supernodes” to highlight the model’s broader computational steps.


## Key findings patterns
- The model reasons across steps (e.g., “capital of the state with Dallas” → “Texas” → “Austin”)
- It plans rhymes when writing poetry
- It generalizes addition to new formats
- It uses medical inference chains to suggest diagnoses
- It distinguishes familiar from unfamiliar knowledge to decide when to answer
- It can be fine-tuned to reject harmful requests—or pursue secret goals covertly

## Resources  
[On the Biology of a Large Language Model](https://transformer-circuits.pub/2025/attribution-graphs/biology.html)

[Circuit Tracing: Revealing Computational Graphs in Language Models](https://transformer-circuits.pub/2025/attribution-graphs/methods.html#graphs-interventions)