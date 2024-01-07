---
title: "[Summary] RAVE: Randomized Noise Shuffling for Fast and Consistent Video Editing with Diffusion Models"
date: 2024-01-06
tags: 
    - Video editing
    - Diffusion models
draft: false 
---

## TL;DR
The process of video editing can be time-consuming and laborious. Many diffusion-based models for videos either fail to preserve temporal consistency or require significant resources. This paper introduces RAVE, a novel video editing method that leverages a pre-trained diffusion model, eliminating the need for additional training.

![Teaser](/posts/20240106_rave/teaser.png)


## Method
To use the methods, two key building blocks are required.

**ControlNet.** A control mechanism for diffusion models by integrating a weight locking strategy, allowing for additional conditions beyond the text prompt (depth, pose, and surface normals). 

**The grid trick (character sheet).** Feeding the diffusion models with a grid of images. The resulting output maintains the grid format during the editing process, ensuring consistent styles.
![Grid trick](/posts/20240106_rave/grid_trick.png)



&nbsp;

__Video editing steps__
1. Extract video frames and apply the condition pre-processor (similarly to ControlNet).
2. Apply video2grid(): Creating an image using a grid of NxN frames (The authors used 2x2 and 3x3 frames). This grid is applied both for the conditioning frames (Condition Grids) and latent diffusion images (Latent Grid).
3. De-nosing steps. For t in 1 to T:
    1. Apply shuffle(): Randomly rearranging the locations of the frames across the grids. This encourages spatio-temporal interaction between the frames.
    2. Feeding the grids, Condition Grid and Latent Grid, to the Unet model.

![Method](/posts/20240106_rave/method.png)


### Design choices
Applying the grid trick directly to videos naively (create a giant big grid) cannot fit the GPU therefore the sub-grids approach is memory efficient.

### Additional novelty
The paper also provides a video evaluation dataset the includes object-focused scenes, human activities like dancing and typing, and dynamic scenes featuring swimming fish and boats.

## Limitations

* Using the grid trick, the output image is with lower resolution. To up-scale it, there a need for additional compute (usually done be a different convolution network).

* The authors say that that the applied the initial grid and preserve the original frame order. This seems unnecessary since right after that the shuffle function is applied.


## Resource
[Arxiv](https://arxiv.org/pdf/2312.04524v1.pdf)

[Project past](https://rave-video.github.io/)

[Github](https://github.com/rehg-lab/RAVE)

[Denoising diffusion implicit models (DDIM)](https://arxiv.org/abs/2010.02502): A method of accelerating the sampling process of diffusion model.

<!-- A good source for catalog videos -->
<!-- https://www.superdry.com/products/details/241523/products?sku=208221850049712A017
https://www.superdry.com/products/details/242208/products?sku=2082218500496V0V030
https://www.superdry.com/products/details/247785/products?sku=1020202000477CQ4004
https://www.superdry.com/products/details/247332/products?sku=102020200051702A002
https://www.superdry.com/products/details/105033/products?sku=2123632000114YM9502
https://www.superdry.com/products/details/205975/products?sku=2082218500449CQ4030
https://www.superdry.com/products/details/244799/products?sku=20822185005025BR019
https://www.superdry.com/products/details/231872/products?sku=210333000017302A017
https://www.superdry.com/products/details/196870/products?sku=217942220004585C033
https://www.superdry.com/products/details/247854/products?sku=208221850051702A019
https://www.superdry.com/products/details/242572/products?sku=214423650046802A020
https://www.superdry.com/products/details/241490/products?sku=2144236000644MD5030
https://www.superdry.com/products/details/197421/products?sku=102020200041602A002
https://www.superdry.com/products/details/89701/products?sku=210322750028807Q019
https://www.superdry.com/products/details/178945/products?sku=2082218500378XVZ020 -->