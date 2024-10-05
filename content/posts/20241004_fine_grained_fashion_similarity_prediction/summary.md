---
title: "[Summary] Fine-Grained Fashion Similarity Prediction by Attribute-Specific Embedding Learning" 
date: 2024-08-24
tags: 
- Fine grained classification
- Transformer
- Fashion
draft: false 
---

## TL;DR
<!-- With discussions about ShopNet 2025, we get inspiration from recent research. One interesting paper is ``Fine-Grained Fashion Similarity Prediction by Attribute-Specific Embedding Learning'':  -->
In the fashion domain, visually distinct products may share fine-grained attributes like sleeve length or collar shape. Traditional methods of finding similar products often overlook these details, leading to irrelevant results for the user. 
To address this, the authors propose a model with two branches: a global branch that processes the entire image and a local branch takes a Region Of Interest (ROI) of specific attributes, identified through a spatial attention layer in the global branch. This approach improves the retrieval of items with similar attributes.

![Retrieved products](/posts/20241004_fine_grained_fashion_similarity_prediction/retrieved_products.png)

## Method
The model architecture consist of two branches: a global branch and a local branch. 

![Model architecture](/posts/20241004_fine_grained_fashion_similarity_prediction/model_architecture.png)


**Global branch.** Takes the whole image as the input and produces ROI of pre-defined attributes.
There are two attentions types: attribute-aware spatial attention (ASA) and attribute-aware channel attention (ACA). The latter is required since same regions may still be related to multiple attributes (e.g. collar design and collar color), 
With both ASA and ACA, the global branch is able to locate the related regions and capture the essential patterns w.r.t. the specified attribute.
 
**Local branch.** Takes a zoomed-in region-of-interest (RoI) w.r.t. the specified attribute of the given image as the input. This allows the local
branch for extracting more fine-grained features. The RoI is obtained by looking at the highly activate regions of the attribute-aware spatial attention of the global branch. 

Notice the local branch is dependent on the global branch, which resorts to the attribute- aware spatial attention of the global branch and regards the highly activated region of the attention map as the RoI with the given attribute.

![Localized Attributes](/posts/20241004_fine_grained_fashion_similarity_prediction/localized_attributes.png)

**Training procedure.** The aim is to have multiple attribute-specific embedding spaces where the distance in a particular space is small for images with the same specific attribute value, but large for those with different ones, i.e.products with the same Round Neck are close to each other in this attribute embeddings space but far away from those with V Neck. 
This is achieved by the triplet ranking loss which applied to both the global and local branches. 

The training procedure is done with two steps: first the global branches is trained until a reasonable ROI is produces. Then the global and local branches are jointly trained in addition to an alignment loss between the global branch and the local branch, which encourages original
images and the corresponding localized regions to be consistently embedded


## Limitations 
Attributes are pre-defined: this requires re-training when introducing an additional attribute

Embeddings are per attribute. The local feature embeddings are not "fused" somehow to the global embeddings. In product retrieval system, this requires indexing each embeddings separately leading to a large memory footprint. 


## Resource
Published in IEEE TRANSACTIONS ON IMAGE PROCESSING, MARCH 2021 
[arxiv](https://arxiv.org/pdf/2104.02429)
