---
title: "From DETR to RF-DETR: The Evolution of End-to-End Object Detection"
date: 2025-10-31
tags:
- Object Detection
- Computer vision
- Real-Time Inference
- Vision Transformer
draft: false
---

## TL;DR
Object detection has shifted from heavy, hand-engineered pipelines based on anchors and heuristics to end to end transformer architectures that learn object localization and classification jointly. This progression (from DETR 2020 to RF-DETR 2025) has reduced post-processing, improved training stability, and brought real-time inference within reach.

![summary_scope)](/posts/20251031_dino_detr/summary_scope.png)

## [DETR: End-to-End Object Detection with Transformers (2020)](https://arxiv.org/abs/2005.12872)

DEtection TRansformer (DETR) introduced a simple yet novel idea: formulate object detection as a direct set prediction problem solved with transformers. 
Instead of relying on multi-stage heuristics such as Anchor Boxes (redefined bounding boxes of specific sizes and aspect ratios spread uniformly over an image) or non-maximum suppression (a heuristic step that removes duplicated detections by suppressing overlapping boxes with lower confidence), DETR learns to predict all objects and their bounding boxes in a single forward pass.

![Anchor boxes](/posts/20251031_dino_detr/anchor_boxes.png)

**DETR Architecture:**
1. The input image passes through a model backbone, extracting a set of dense feature vectors.
2. These features are fed into a transformer encoder, which applies multi-head self-attention. 
3. Positional encodings are added to maintain spatial information since transformers are permutation invariant.
4. The transformer decoder receives a fixed number of learnable object queries. Each query predicts an object or no object, and the decoder’s cross-attention layers relate queries to encoded image features. *Notice this means the number of object queries is pre-defined and needs to be larger than the largest #object in the dataset*.
5. All objects are predicted simultaneously in a single forward pass, removing the need for post-processing like NMS.


![detr_architecture](/posts/20251031_dino_detr/detr_architecture.png)

**The DETR loss function** is a two-step process:
1. Hungarian (bipartite) matching: Finding the best unique assignment between predicted objects and ground truth by minimizing a cost combining classification confidence and bounding box similarity.
2. After matching, a standard loss is computed on the matched pairs: (i) Classification loss (cross-entropy between predicted and true class labels) and (ii) Bounding box loss (a combination of L1 and IoU loss).

![hungarian_matching](/posts/20251031_dino_detr/detr_hungarian_matching.png)

DETR was a breakthrough by removing the need for manual steps with anchor boxes and NMS, however, it trains slower than traditional methods and matches their accuracy without clearly beating them.

## [DINO: DETR with Improved DeNoising Anchor Boxes (2022)](https://arxiv.org/pdf/2203.03605)
*Notice Not related to Meta’s DINOv2 or DINOv3.*

DETR with Improved DeNoising Anchor Boxes (DINO) built directly upon DETR, aiming to fix its main issues: slow convergence and poor small object detection detections. It introduced a few architectural and training refinements that helped DETR-like models become faster, more accurate, and easier to train.

![dino_architecture_simplified](/posts/20251031_dino_detr/dino_architecture_simplified.png)

DINO's Innovations:
1. Contrastive Denoising Training: DINO generates both positive (slightly perturbed) and hard negative (noisier) box queries from ground truth, explicitly teaching the model to reconstruct objects from positives and reject negatives. This minimizes duplicate predictions and improves query-to-object assignment stability.

2. Mixed Query Selection: DINO refines query initialization by splitting spatial and semantic components. Instead of deriving all queries from learned embeddings as in DETR, it uses encoder-predicted box coordinates to initialize query positions, while keeping the content embeddings fixed and learned separately.
![mixed_query_selection](/posts/20251031_dino_detr/mixed_query_selection.png)

3. Look-Forward-Twice Mechanism: Intermediate decoder layer outputs are supervised both directly and via gradient flow from later layers

DINO set a new standard in transformer-based object detection by elegantly solving DETR’s bottlenecks. 
Its core ideas—learned anchors, contrastive matching, and improved gradient flow—now are used in any modern object detectors.. 

## [RF-DETR: A SOTA Real-Time Object Detection Model (2025)](https://blog.roboflow.com/rf-detr/)
Roboflow Detection Transformer (RF-DETR) is a transformer-based object detector focused on real-time performance, achieving state of the art performance (over 60 mAP on the COCO dataset) while running at 25 FPS on NVIDIA T4 GPUs.

Main novelties:
1. Deformable Attention: Each feature only attends to a sparse, learned set of spatial keypoints instead of all pixels, reducing quadratic complexity.
2. Multi-scale Sampling: Aggregates information from multiple feature map levels to better capture small objects.
3. Latency-Aware Transformer: Reduces decoder depth and attention redundancy guided by inference profiling, maintaining performance with lower computational cost.
4. Modern Pretraining: Adopts a DINOv2 backbone pretrained through large-scale self-supervision, enhancing transfer and low-data adaptation.

RF-DETR represents the cutting edge of real-time object detection by uniting deformable attention, efficient transformer design, and powerful pre-training. 
It delivers state-of-the-art accuracy with the speed needed for practical deployment.

![rfdetr_architecture](/posts/20251031_dino_detr/rfdetr_architecture.png)

## Reference

Traditional Object Detection:
- [Object Detection — Anchor Box VS Bounding Box](https://medium.com/@nikitamalviya/object-detection-anchor-box-vs-bounding-box-bf1261f98f12)

DETR (DEtection TRansformer) 2020:
- [End-to-End Object Detection with Transformers (arXiv)](https://arxiv.org/pdf/2005.12872)
- [verview of DETR Architecture and Training](https://dzdata.medium.com/detr-end-to-end-object-detection-with-transformers-f40ce77bfe44)

DINO DETR (DETR with Improved DeNoising Anchor Boxes)  2022:
- [DINO: DETR with Improved DeNoising Anchor Boxes (arXiv)](https://arxiv.org/abs/2203.03605) 
- [How to DINO-DET with Improved Denoising Anchor Boxes (Medium article)](https://akmaier.medium.com/how-to-dino-detr-with-improved-denoising-anchor-boxes-d9799f024773)

RF-DETR (Roboflow Detection Transformer) 2025:
- [RF-DETR: Real-Time Transformer Detection ](https://blog.roboflow.com/rf-detr/)
- [Insights into RF-DETR architecture (Medium article)](https://towardsdatascience.com/rf-detr-under-the-hood-the-insights-of-a-real-time-transformer-detection/)
- [LW-DETR: A Transformer Replacement for YOLO for Real-Time Detection (arXiv)](https://arxiv.org/pdf/2406.03459)

[Excalidraw figures](https://excalidraw.com/?#json=OPo23HDKhm_iS1Ul3e7by,XwM7kYYPfgGyYoo5iXXxxA)