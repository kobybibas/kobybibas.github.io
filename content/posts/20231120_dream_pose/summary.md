---
title: "[Proof of Concept] DreamPose: Fashion Image-to-Video Synthesis via Stable Diffusion"
date: 2023-10-14
tags: 
draft: true
---

## TL;DR
Generating animated fashion videos from still images based on input image and pose sequence (represented by the DensePose model). Fashion videos refer to a human model animated with a pose sequence.

![Example](/posts/20231120_dream_pose/pose_sequence.png#center)

## Problem statements
Diffusion models able to generate images based on given text. However, they do not produce animated sequence nor able to be conditioned on an input pose sequence.


## Method 
Taking a diffusion model and apply the following changes:
1. Insread of the CLIP text encoder, replace by CLIP image encoder + VAE encoder + Adapter
2. Along with a random noise, feed the UNET model a pose sequence
3. 


![Architecture](/posts/20231120_dream_pose/architecture.png#center)

## Limitations
Each input image requires finetunining of the UNET, adapter and VAE. On V100 it takes  ~1 hour.



## Resource
<https://grail.cs.washington.edu/projects/dreampose/>
