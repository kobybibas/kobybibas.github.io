---
title: "[Summary] Unifying Generative and Dense Retrieval for Sequential Recommendation" 
date: 2025-01-04
tags: 
- Recommender system
- Large Language Models 
- Multimodal 
draft: false 
---

## TL;DR 
Traditional item retrieval methods use user and item embeddings to predict relevance via inner product computation, which is not scalable for large systems. Generative models predict item indices directly but struggle with new items. This work proposes a hybrid model that combines item positions, text representations, and semantic IDs to predict both the next item embedding and several possible next item IDs.
Then only this item subset along the new items are in the inner product with user representations.    

## Generative Retrieval
To train a generative model for predicting the next semantic item ID:
1. Collect textual descriptions for each item to generate text embeddings, which are quantized into semantic IDs using an RQ-VAE.
2. Replace item IDs in interaction histories with their semantic IDs.
3. Train a Transformer model to predict the next semantic ID sequence based on the last \(n\) items a user interacted with.
4. Use beam search during inference to retrieve candidate items based on user interactions.

![Semantic ID generation](https://blog.reachsumit.com/img/posts/2023/llms-for-recsys-entity-representation/semantic_id_generation.png)

## Comparison between Dense and Generative Retrieval 
Dense retrieval uses maximum inner product search, while generative retrieval uses next-token prediction via beam search. Key differences include:
1. Dense retrieval requires O(N) embeddings for \(N\) items, whereas generative retrieval needs O(t) tokens for user interactions.
2. The effectiveness of semantic IDs in capturing item semantics is uncertain.
3. Generative retrieval performs poorly with cold-start items due to overfitting to training data.

![Dense_vs_generative_retrieval](/posts/20250104_unifying_generative_and_dense_retrieval_for_sequential_recommendation/dense_vs_generative_retrieval.png)

## Method for Combining Dense and Generative Retrieval
The SID-based hybrid model uses dense retrieval on a limited set of candidates generated by generative retrieval, improving performance while maintaining minimal storage requirements.

The hybrid model approach is as follows. 
1. Construct input embeddings by combining positional encoding, semantic ID, and item text representation.
2. Train a model with cosine similarity loss and next-token prediction loss on the next item's semantic IDs. This model produces both embeddings and semantic ID.
3. During inference, retrieve \(K\) items via beam search, supplement with cold-start items, and rank using the encoder’s output embeddings.

![Architecture](/posts/20250104_unifying_generative_and_dense_retrieval_for_sequential_recommendation/architecture.png)

## Limitations 
1. The assumption that cold-start items are sparse may not always hold.
2. Regular training on new items is necessary for the next SID predictor.
3. The hybrid model's complexity may increase inference time, impacting real-time recommendation performance.

## Resource
[arxiv](https://arxiv.org/pdf/2405.17927)

[Representing Users and Items in Large Language Models based Recommender Systems](https://blog.reachsumit.com/posts/2023/06/llms-for-recsys-entity-representation/)