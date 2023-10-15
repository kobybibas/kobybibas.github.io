---
title: "Summary: Drag Your GAN: Interactive Point-based Manipulation on the Generative Image Manifold"
date: 2023-10-14
tags: 
    - GAN
    - Image editing
---

**TL;DR** This work enables interactive editing of a GAN's generated image by translating ("dragging") any point in the image to a target location.

## Problem statements
GAN based image generation takes a noise vector to generate an image. There is a need of a localized controlled image manipulation as moving a region to a different location in the image.

## Method 
Given a GAN generated image, a user input of the source coordinates (q) and the coordinates of the destination (p)
1. Project the coordinates to the GAN feature space (the 6-th layer) (F(q),F(p)).
2. Calculate the direction vector between the source and destination F(q) -> F(p).
3. Finetune the GAN layers (up to the 6th layer) such that the features at the source coordinates + small translation toward the destination coordinates be equal to the source features.
4. Repeat 1-3 until reaching the destination location.

An additional contribution is: instead using a tracker to track the movement of the source features toward the target destination that might add an additional computation cost, they track the movement of the source features directly using nearest neighbor with the GAN's feature space.

## Limitations
This method enables the manipulation of a GAN's generated image only thus a user cannot feed it their own images.

<figure>
    <img src="/images/drag_your_gan.png"
         alt="Method architecture"
         width="764">
</figure>


<https://arxiv.org/abs/2305.10973>