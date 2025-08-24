---
title: "[Summary] Relightable Gaussian Codec Avatars"
date: 2025-02-28
tags:
  - Codec Avatar
  - Gaussian Splatting
  - Conditional Variational Autoencoder
  - Relighting
draft: false 
---

## TL;DR 
Photorealistic head avatars is a required technology for virtual and augmented reality.
However, current approaches either lack the fidelity to capture fine details (like hair) or are too slow for real-time use.
Relightable Gaussian Codec Avatars proposes using (i) 3D Gaussian splatting for efficient geometry representation and (ii) a learnable radiance transfer model for appearance, including an explicit eye model.
![teaser](/posts/20250228_relightable_gaussian_codec_avatars/teaser.png)


# Background 
Image relighting is the task of showing what a scene from a source image would look like if illuminated differently.
The fidelity of relighting is bounded by both geometry and appearance representations. 
1. For geometry, standard approaches of mesh and volumetric approaches have difficulty modeling fine structures like 3D hair geometry. 
2. For appearance, existing relighting models are limited in fidelity and often too slow to render in real-time with high-resolution continuous environments.

**Gaussian splatting.** Given multi-view images of a scene:
1. Extract point cloud
2. Replace each point with a Gaussian defined by its translation, per-axis scale, color, and opacity.
3. Optimize the Gaussian parameters via gradient descent to accurately reconstruct the scene. Adaptively split, merge, or clone Gaussians to capture fine details.
With this scene representation, we can render different view-points in real time by considering only the Gaussians influencing each pixel from any viewpoint.

![Gaussian splatting](https://miro.medium.com/v2/resize:fit:720/format:webp/0*rT4bEjhOZyCUqBuR)


## Method
The proposed method comprises two components, one to tackle the geometry representation and the other for the avatar appearance
![method_overview](/posts/20250228_relightable_gaussian_codec_avatars/method_overview.png)


### Geometry Representation: Gaussian Splatting
The face is modeled as a collection of 3D Gaussians defined by their position, orientation, scale, opacity, and color—which enable efficient rendering through splatting techniques. Differently from the standard task of Gaussian splatting, Codec Avatars has these challenges:
* Dynamic Expressions: The face changes along the video, so the representation must capture dynamic expressions and movements.
* Lighting Variations: Changing illumination requires the ability to re-color the scene in real time.

**Solution Overview:**
1. Shared UV Texture Mapping: All Gaussians are parameterized on a shared UV texture map
2. A conditional variational autoencoder (CVAE) processes a coarse 3D mesh to extract a latent vector representing facial expressions.
3. Using the latent code and eye gaze directions, a geometry decoder outputs adjustments—such as position offsets, rotation, scale, and opacity—for each Gaussian.
4. A separate decoder refines the overall mesh
5. The final Gaussian positions are computed by combining the adjustments (step 3) with interpolated positions from the overall mesh (step 4).


### Appearance: Learnable Radiance Transfer
The appearance model splits the color of each 3D Gaussian into two parts:
1. A view-independent diffuse term: The diffuse term uses spherical harmonics to efficiently capture global light effects—like occlusion, subsurface scattering, and multi-bounced scattering in hair—providing a robust low-frequency lighting representation. 
2. A view-dependent specular term: The specular term leverages spherical Gaussians to model sharp, high-frequency reflections on surfaces like skin and eyes. An integrated explicit eye model further refines eye reflections and allows independent gaze control. 
Together, these components maintain the linearity of light transport, enabling accurate, real-time relighting under both point lights and environment map illumination.


### Data collection and Training
To train the model, the data collection effort included 110-camera multi-view setups with 144K frames per participant performing varied facial actions with different illuminations
Given multi-view video data of a person illuminated with known point light patterns, all trainable network parameters were jointly optimized.


## Limitations
1. The setup used for capturing training data is highly specialized (with high-resolution multiview cameras and a complex light stage), which may limit the method’s accessibility for in-the-wild applications.
2. The approach assumes known illumination conditions; inaccuracies in illumination estimation may reduce the overall photorealism.
3. Although the model captures detailed expressions, extreme facial deformations or occlusions may still pose challenges for the geometric reconstruction.


## Resources
[arxiv](https://arxiv.org/abs/2312.03704), CVPR 2024 (Oral)

[Project page](https://shunsukesaito.github.io/rgca/)

[3D Gaussian Splatting! - Computerphile](https://www.youtube.com/watch?v=VkIJbpdTujE)