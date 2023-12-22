---
title: "[Proof-of-Concept] DreamPose: Fashion Image-to-Video Synthesis via Stable Diffusion"
date: 2023-11-18
tags: 
    - Video generation
    - Video editing
    - Animation
    - Diffusion model
    - Image to video
draft: false
---

## TL;DR
Typical diffusion models create images using input text. [DreamPose](<https://grail.cs.washington.edu/projects/dreampose/>), presented at ECCV 2023, enhances this functionality by generating a video from an image incorporating a human model and pose sequence, as represented by [DensePose](http://densepose.org/).

![Example](/posts/20231120_dream_pose/pose_sequence.png#center)

## Problem statements
Common diffusion models able to generate images based on given text. However, they can not produce animated sequence nor able to be conditioned on an input pose sequence.

## Method 
Apply the following modifications to a diffusion model:
1. Substitute the CLIP text encoder with a combination of CLIP image encoder + VAE encoder + Adapter.
2. Concatenate a pose sequence, along with random noise, to the UNET model.

The rationale for step 1: Conditioning by an image can be done by concatenating the image with input noise for the denoising U-Net(as in  [InstructPix2Pix](https://www.timothybrooks.com/instruct-pix2pix)). However, the goal is to generate images that are *not* spatially aligned with the input image. For step 2, the pose conditioning is aligned with the image. Consequently, the noisy latents can be connected to the target pose representation.

The training procedure comprises the following steps:
1. Initialize the unaltered Stable Diffusion layers from pre-trained text-to-image Stable Diffusion checkpoints.
2. Fine-tune the UNet and adapter module on the complete training dataset ([UBC Fashion](https://vision.cs.ubc.ca/datasets/fashion/)) to generate frames consistent with an input image and pose.
3. Further fine-tune the UNet and adapter module, followed by the VAE decoder, using one or more subject-specific input images to create a model tailored to the specific subject.
4. During inference, generate video frames frame-by-frame from a single input image and a sequence of poses using the subject-specific model.
 
![Architecture](/posts/20231120_dream_pose/architecture.png#center)

# Dataset
The model is trained in step 2 with the [UBC Fashion dataset](https://vision.cs.ubc.ca/datasets/fashion/). It contains 339 training and 100 test videos. Each video has a frame rate of 30 frames/second and is approximately 12 seconds long. During training, the authors randomly sample pairs of frames from the training videos.

## Limitations
* Fine-tuning the UNET, adapter, and VAE is necessary for each input image, and on V100, takes approximately one hour.
* The domain is highly constrained: only images featuring humans as inputs and producing solely different poses of human models as outputs.
* I tried running the code with Zara images. Results seem not as good as the paper figures. 


# Proof-of-Concept
Testing the method's performance by Zara image. \
On the Left: the input image, on the Right: the generated video.
{{< video label="output_zara2" mp4="/posts/20231120_dream_pose/output_zara2.mp4" >}}
{{< video label="output_zara3" mp4="/posts/20231120_dream_pose/output_zara3.mp4" >}}
The temporal consistency and the visual similarity to the original image are not great. The reason might be that I fine-tuned the model using "--use_8bit_adam" which might produce lower quality videos than the original implementation.


## Resource
[Paper webpage](<https://grail.cs.washington.edu/projects/dreampose/>)

<https://arxiv.org/abs/2304.06025> published in ICCV 2023

[Original github repository](https://github.com/johannakarras/DreamPose)

[My adaptation of the github repository](https://github.com/kobybibas/DreamPose)