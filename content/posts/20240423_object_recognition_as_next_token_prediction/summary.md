---
title: "[Summary] Object Recognition as Next Token Prediction"
date: 2024-04-23
tags:
  - Large Language Models
  - Object Recognition
  - Prompt Engineering
  - Open Set Recognition
  - Summary
draft: false 
---

## TL;DR
Models for object classification require a fixed set of pre-defined classes which constrain the model from recognizing any object. In this paper, a visual classifier is trained to predict the most likely token of a pre-trained Large Language Model (LLM). 
Given that LLMs are trained on extensive textual data, training a model to predict across the entire token space allows it capture the full range of textual information.

![teaser](/posts/20240322_object_recognition_as_next_token_prediction/teaser.png)


## Methods
The model is trained to predict the probability for each token of a pretrained LLM:
Denote Xv as the visual features, W as the LLM token embeddings, and w represents the most probable single token, the model prediction is
![token_prediction](/posts/20240322_object_recognition_as_next_token_prediction/token_prediction.png)

To guide the language decoder, the authors prompt it with “the objects in the image are” (Xp). 
Then they concatenate the visual features (Xv) and instruction (Xp) along with the special token [IMG] to indicate the boundary 
![teaser](/posts/20240322_object_recognition_as_next_token_prediction/decoder_tokens.png)


**Multiclass prediction.**
Typically, a label consists of multiple tokens, e.g., “sofa” has two tokens [so] and [fa]. 
To predict multiple tokens, the following procedure is applied.
1. The input tokens (Xv + [IMG] + Xp) are propagated to the decoder.
2. The tokens are rank by the softmax probabilities and only the top k are kept.
3. These top k tokens are used to condition the next token predictions.
4. Steps 2-3 are repeated until the [SEP] token (that indicates the end of the class) has the highest softmax probability.

![teaser](/posts/220240322_object_recognition_as_next_token_prediction/multiclass_sampling.png)

**Loss function.** The model is train on image-caption pairs and extract nouns from raw captions as reference labels to weakly supervise training. The loss function is the cross-entropy loss.


## Resource
[Arxiv](https://arxiv.org/pdf/2312.02142) Published in CVPR 2024

