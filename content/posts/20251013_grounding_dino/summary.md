---
title: "[Summary] Grounding DINO: Marrying DINO with Grounded Pre-Training for Open-Set Object Detection"
date: 2025-10-13
tags:
- Object Detection
- Vision-Language
- Open-Set
draft: true
---

## TL;DR

## Motivations

## Method

Grounding dino is based on dino detr and not original dino
18 S. Liu et al.
57. Zhang, H., Li, F., Liu, S., Zhang, L., Su, H., Zhu, J., Ni, L.M., Shum, H.Y.: DINO:
DETR with improved denoising anchor boxes for end-to-end object detection (2022)


Tightly couple text and image in many places in the architecture.

Given words, assign bounding boxes to each of th words. The words act as the label

Grounding DINO is a dual-encoder-single-decoder architecture. It contains
an image backbone for image feature extraction, a text backbone for text feature
extraction, a feature enhancer for image and text feature fusion (Sec. 3.1), a
language-guided query selection module for query initialization (Sec. 3.2), and a
cross-modality decoder for box refinement (Sec. 3.3)

Training with 16 Nvidia v100 gpu for tiny and 64 a100 for large model 



![Architecture](/posts/20251013_grounding_dino/architecture.png) 


## Limitations

## Resource
(arXiv)[https://arxiv.org/abs/2303.05499]
