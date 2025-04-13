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
Large Language Models (LLMs) are often considered “black boxes,” making it hard to explain their predictions or assess whether they genuinely reason. 
A local “replacement model” approach swaps original MLP layers with simpler linear modules while preserving most of the behavior. 
By mapping out how features influence one another, structured reasoning, planning, and even hidden goals are revealed in the LLM’s computations.

## Method
One reason models are difficult to interpret is that neurons are typically polysemantic: they perform many different, seemingly unrelated functions. 
The aim is to produces a “replacement model” that behaves similarly to the original network but is much easier to interpret.
To achieve the goal, the LLM is applied with these changes:
1. For a given prompt, fix the attention layer
2. Swapping MLP layers with linear approximations of pre-defined explainable features  
3. Adding error nodes to have more similar prediction to the original LLM
4. Creating attribution graph by pruning node


**Cross-Layer Transcoders (CLTs)** replicate the behavior of each MLP in the transformer by using linear approximations of earlier layer activations. The process involves:
1. Running a forward pass of the original transformer to record inputs and outputs from each MLP layer
2. Fix the attention layer
3. Training a lightweight linear module to approximate each MLP’s mapping from inputs to outputs
4. Employing a simple regression objective (e.g., minimizing mean squared error) to align the CLT’s outputs with those of the original MLP

**Error Nodes.** Since MLP layers are replaced by linear approximations, they cannot fully reconstruct the original model’s outputs. 
To increase representational capacity, error nodes are introduced to capture the residual information that the linear layers fail to account for.

![replacement_model](/posts/20500412_biology_large_language_model/replacement_model.png)

**Attribution Graphs.** A directed graph is constructed from the replacement model as follows:
* Nodes: represent active features, input tokens, or outputs.  
* Edges: represent how strongly one feature linearly contributes to another.  

The process traces from input to output through active features, pruning paths with negligible impact. 
Related features often form groups with shared roles, combining these into “supernodes” highlights broader computational steps in the model.

![Attribution graph](/posts/20500412_biology_large_language_model/attribution_graph.png)

## Uncovered patterns
- The model reasons across steps (e.g., “capital of the state with Dallas” → “Texas” → “Austin”)
- It plans rhymes when writing poetry
- It generalizes addition to new formats
- It uses medical inference chains to suggest diagnoses
- It distinguishes familiar from unfamiliar knowledge to decide when to answer
- It can be fine-tuned to reject harmful requests—or pursue secret goals covertly

## Resources  
[On the Biology of a Large Language Model](https://transformer-circuits.pub/2025/attribution-graphs/biology.html)

[Circuit Tracing: Revealing Computational Graphs in Language Models](https://transformer-circuits.pub/2025/attribution-graphs/methods.html#graphs-interventions)