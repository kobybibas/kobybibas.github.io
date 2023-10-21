---
title: "[Summary] MultiDiffusion: Fusing Diffusion Paths for Controlled Image Generation"
date: 2023-05-19
tags: 
    - Diffusion model
    - Image editing
    - Controllability 
---

## TL;DR
To enable a more controllable image diffusion, MultiDiffusion introduce patches generation with a global constrain.

## Problem statements
Diffusion models lack user controllability and methods that offer such control require a costly fine-tuning.

## Method 
The method can be reduced to the following algorithm:
At each time step t:
1. Extract patches from the global image I_{t-1}
2. Execute the de-noising step to generate the patches J_{i,t} 
3. Combine the patches by average their pixel values to create the global image I_t

For the panorama use case: simply generate N images with overlapping regions between them. At each de-noising step, take the average pixel values of the overlapping regions.

## Limitations
The compute cost is linear in the number of patches (each additional patch require diffusion model inference)

<figure>
    <img src="/content/posts/20230525_multidiffusion/process.png"
         alt="Image generation process"
         width="764">
</figure>
<figure>
    <img src="/content/posts/20230525_multidiffusion/examples.png"
         alt="Generated image example"
         width="764">
</figure>

# Resource
<https://arxiv.org/abs/2302.08113>