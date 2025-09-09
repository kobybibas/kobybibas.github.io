---
title: "[Summary] Bag of Tricks for Multimodal AutoML with Image, Text, and Tabular Data"
date: 2025-09-09
tags:
- AutoML
- Multimodal Learning
- Image Processing
- Text Analysis
- Tabular Data
draft: true
---

## TL;DR
Automatic machine learning (AutoML) aims to automate the process of building ML
models, including data preprocessing, model selection, hyperparameter tuning, model evaluation, and so on.
Automated Machine Learning (AutoML) systems often focus on a single data modality, limiting their applicability to real-world problems that involve a mix of images, text, and tabular data. 

![multimodal example](/posts/20250909_bag_of_tricks_multimodal_automl/multimodal_example.png)This paper analyze different design choices for building effective AutoML pipelines for multimodal datasets.


Key aspects of multimodal learning include 
1. determining the best multimodal fusion strategies 
   - early vs late fusion  
   - parallel vs. sequential fusion
2. conducting data augmentation for multimodal data 
   - independent vs. joint augmentation
   - input vs. feature augmentation
3. assessing the usefulness of cross-modal alignment for supervised learning, 
4. handling samples with missing modalities

For each aspect, we investigate multiple strategy alternatives. Rather than proposing new techniques, our goal is to gather existing ones from various literatures and codebases and conduct an empirical evaluation in a novel yet practical setting with flexible mixtures of image, text, and tabular modalities.

## Motivation
Central to an
AutoML system is a collection of "tricks" that significantly enhance performance.


We scrutinize design choices related to multimodal fusion strategies, multimodal data augmentation, converting tabular data into text, cross-modal alignment, and handling missing modalities. 

## Learnings

**Baseline.** Employs a late-fusion architecture  to independently process the image, text, and tabular
data. It uses 
* Swin Transformer Large,
* DeBERTa-V3 [33 ], 
*  FT-transformer [28 ] 
These are followed by a multilayer perceptron (MLP) for feature fusion and a linear head for prediction.


## Basic tricks
1. Greedy soup: Averaging the weights of multiple fine-tuned models, known as model
soup, often improves accuracy and robustness. This paper applies greedy soup to the top 3
checkpoints along a single training trajectory.

2. Gradient clipping: This technique addresses the exploding gradient problem by
limiting the magnitude of the gradients, preventing them from growing unchecked during
training.
3. Learning rate warmup : Starting with a small learning rate and gradually increasing it
over a few initial epochs or iterations helps prevent the model from diverging during the
initial phase of training.

4. Layerwise learning rate decay: It allocates distinct learning rates to each layer, with shallower
layers assigned higher learning rates and smaller rates for deeper layers. T


We find that the model average performance improves significantly when employing greedy soup, gradient clipping and learning rate warmup. The layerwise decay has mixed, yet generally positive, effects when considering all the modality combinations. 


### Multimodal Fusion Strategies


Late-fusion strategies generally outperform early-fusion.
Among the late fusion methods, LF-Aligned significantly outperforms other variants
across different modality compositions, except for the text+tabular composition


* LF MLP: Uses a separate model for each modality followed by an MLP module is used for fusing the concatenated CLS [17] token features.
* LF Aligned: Follows the LF-MLP setup but leverages CLIP’s image and text encoders, which have been aligned through large-scale contrastive pre-training.


Interestingly, some tricks, although not improving performance on their own, contribute positively to
the ensemble, highlighting the complementary nature of different tricks.


### Converting Tabular Data into Text

tabular serialization—converting
tabular data into text—in multimodal settings remains relatively under-explored.

- Converting numeric values into text leads to a negative performance effect compared to Baseline+.
- converting categorical values can enhance model performance in most cases, except for the image+text+tabular composition. 
We attribute this improved performance to the similarity between categorical and text
data. Representing categorical data in text format can directly convey its semantic information,
potentially helping the model better understand the meaning of different categories in most cases.

### Cross-modal Alignment

### Multimodal Data Augmentation

### Handling Modality Missingness

### Overall / ntegrating Bag of Tricks
Our findings indicate that while some tricks consistently perform
well across the entire benchmark, many are effective for certain modality combinations but struggle
to generalize across others. Ultimately, we employ a learnable ensembling method [ 9] to integrate all
the techniques into a single pipeline, resulting in superior performance compared to each individual
technique. 

# Limitations

## Results

## Resource
[Paper](https://arxiv.org/abs/2412.16243)