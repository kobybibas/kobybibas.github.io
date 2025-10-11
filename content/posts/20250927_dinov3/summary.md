---
title: "[Summary] DINOv3: Self-Supervised Vision Transformers at Scale"
date: 2025-09-27
tags:
- Self-Supervised Learning
- Vision Transformers
- Representation Learning
- Computer Vision
draft: false
---

## TLDR

## Motivation

## DINOv1
Trains a student model to match the output of a slowly updated teacher model on multiple views of the same image. The architecture is Vision Transformer (ViT) for both teacher and student. such that they produce a trivial soluton (a fix number)

**Dataset.** Training dataset is ImageNet-1k 


**New training techniques.**
To prevent the trivial collapse of the features (produce a trivial soluton like a fix number), the following measures are implemented:
1. The teacher weights are an exponential moving average (EMA) of the student’s weights
2. The teacher outputs are centered by subtracting the batch mean. 
3. The teacher outputs are sharpened by applying a low softmax temperature, which prevents the distribution from becoming too flat.

## DINOv2
uses a larger scale model (ViTs are data hungry) x100 more data than DINOv1.

**Dataset.**
Created a new dataset LVD-142M (starting from an initial set of ~1.2B web images) of high quality curated images.
* Utilizing curated data sources (ImageNet-22k, the train split of ImageNet-1k, Google Landmarks, and several fine-grained classification datasets)
* Filter near-duplicate images
* Exapnd the curated data by retrieving samples close to the curated source using features from a DINO-like model

![DINOv2_data_processing_pipeline](/posts/20250927_dinov3/DINOv2_data_processing_pipeline.png)

**New training techniques.**

1. A new patch-level loss from iBOT [4] (another SSL method), where some input patches are masked for the student (but not for the teacher). The student is then asked to predict the teacher’s features for those masked patches, encouraging good local representations that complement the global objective. 

2. Different heads for global and patch-level objectives, which separate learnable MLP projections are applied to the [CLS] token for global features and to the regular patches for local features, avoiding interference at scale.
The [CLS] token is placed at the beginning of the input sequence and serves as an aggregate representation that encapsulates information from the entire input sequence. In vision models, this token acts as a global feature representation that has "seen" and processed information from all the image patches through the transformer's self-attention mechanism

3. teacher centering + sharpening is replaced by a special batch normalization 

4.  Mixed-resolution training 


## DINO V3
After DINOv2, a follow-up analysis shoed that transformer smuggles global image information into irrelevant background patches, essentially “upclassing” them to carry global context and creating more room for complex reasoning.

The fix is very simple: add a few register tokens to the input, giving the model a dedicated place to stash global information and keeping patch representations clean. Similar to the [CLS] token, which is not tied to any image patch, these extra tokens help smooth attention maps and improve dense prediction

**Dataset.** Public posts on Instagram,  another x10 on the data side, 1.7B images.
1. To get a balanced diversity: images are first embedded with DINOv2, then clustered and subsampled in a balanced way from each group to ensure coverage of all visual concepts appearing on the internet
2. the retrieval step picks images close to trusted seed datasets to cover visual concepts relevant for downstream tasks

**Architecture changes.**
1. DINOv3 also scales up aggressively. The team introduces a custom ViT-7B architecture, making it the largest vision transformer in this line of work so far (larger variants have been trained for supervised tasks, e.g., by Google).
2. Patch size is increased from 14 to 16, and special attention is also put on positional embeddings, using an improved RoPE variant with box jittering augmentation to later handle different resolutions, scales, and aspect ratios more robustly.

**New training techiniques.**
1. To prevent breaking down of intra-patch consistency a new loss function operates on the Gram matrix (i.e., the matrix of all pairwise dot products of patch features in an image) and pushes the Gram matrix of the student towards that of an early iteration of the teacher.

* Post training on mixed resolutions per mini-batch. Specifically, global crop sizes are sampled from {512, 768} and local crop sizes from {112, 168, 224, 336}. This is since the inital trainig is done on 256x256 image resolution. 
Rope positional encoding to  handle various of resolution 

* aligning DINOv3 with a text encoder, since CLIP-style models with open-vocabulary are key to modern multimodal research. The focus here is on keeping the vision backbone completely frozen, so we can keep all its beautiful properties, and see if a text encoder can be trained from scratch with a contrastive objective to match current state-of-the-art models.



## Applications 

For unsupervised object detection, we use the non-parametric graph-based TokenCut algorithm (

Video segmentation tracking :
given ground-truth instance segmentation masks in the first
frame of a video, the goal is to propagate these masks to subsequent frames. Following Jabri et al. (2020), we use a non-parametric label propagation
algorithm that considers the similarity between patch features across frames. 

Video classification 
we train an attentive
probe—a shallow 4-layer transformer-based classifier—on top of patch features extracted from each frame.
This enables reasoning over temporal and spatial dimensions as the features are extracted independently per


Object detection
Implementation We build upon the Plain-DETR (Lin et al., 2023b), but make the following modification:
We do not fuse the transformer encoder into the backbone, but keep it as a separate module, similar to the
original DETR (Carion et al., 2020), which allows us to keep the DINOv3 backbone completely frozen during
training and inference. To the best of our knowledge, this makes it the first competitive detection model to
use a frozen backbone.
Kb: this suggest generally we should fine tune for object detection 


## Limitations

# Resource
[DINOv1 Paper](https://arxiv.org/pdf/2104.14294)

[DINOv2 Paper](https://arxiv.org/pdf/2304.07193)

[DINOv3 Paper](https://arxiv.org/pdf/2508.10104)

[From DINO to DINOv3](https://mlhonk.substack.com/p/39-from-dino-to-dinov3)

[Vision Transformers Need Registers](https://arxiv.org/pdf/2309.16588v2)

