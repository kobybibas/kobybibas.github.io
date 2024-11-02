---
title: "[Summary] The Evolution of Multimodal Model Architectures" 
date: 2024-11-01
tags: 
- Multimodal
- Visual-Language-Models
draft: false 
---

## TL;DR 
Multimodal models are advancing rapidly across research and industry. Their architecture can be characterized into four different types.
1. Types A and B integrate multimodal data within the internal layers of the model.
    1. Type A relies on standard cross-attention for fusion
    2. Type B introduces custom-designed layers for multimodal fusion
2. Types C and D fuse multimodal at the input stage (early fusion)
    1. Type C uses modality-specific encoders without tokenization
    2. Type D employs tokenizers for each modality at the input and able to generate outputs with multimodalities (any-to-any multimodal models)


## Model Architecture Overview
Models processing images, audio, or video alongside text have evolved significantly. Recent developments have emphasized image-text integration across diverse vision-language tasks, resulting in varied multimodal architectures based on the fusion method.


### Type A: Standard Cross-attention based Deep Fusion (SCDF)
![Type A](/posts/20241101_the_evolution_of_multimodal_model_architectures/type_a.png)

1. Uses a pre-trained language model (LLM)
2. Multimodal inputs (images/audio/video) pass through modality-specific encoders
3. A resampler generates embeddings suitable the requirements of the decoder layer
4. Cross-attention layers facilitate deep fusion of multimodal data within the model’s layers
 
Examples of models in this category: Flamingo, OpenFlamingo, Otter (trained on MIMIC-IT dataset on top of OpenFlamingo), MultiModal-GPT (derived from OpenFlamingo), PaLI-X, IDEFICS (open-access reproduction of Flamingo), Dolphins (based on OpenFlamingo architecture), VL-BART, and VL-T5.

### Type B: Custom Layer based Deep Fusion (CLDF)
![Type B](/posts/20241101_the_evolution_of_multimodal_model_architectures/type_b.png)

1. Employs a pre-trained LLM
2. Modality-specific encoders process the multimodal input
3. Custom cross-attention layers, such as learnable linear layers or MLPs, integrate multimodal data in the model’s internal layers
4. Some implementations include a learnable gating factor to control cross-attention contributions

Examples of models in this category: LLaMA-Adapter, LLaMA-Adapter-V2 G, CogVLM, mPLUG-Owl2, CogAgent, InternVL, MM-Interleaved, CogCoM, InternLM-XComposer2, MoE-LLaVA, and LION.


### Type C: Non-Tokenized Early Fusion (NTEF) 
![Type C](/posts/20241101_the_evolution_of_multimodal_model_architectures/type_c.png)

This approach stands out as the most widely adopted multimodal model architecture.

1. A pretrained LLM acts as the decoder without architecture modifications.
2. Modality-specific encoders are employed.
3. Multimodal fusion occurs at the input stage (without tokenization, a non discrete inputs are fed to the model).

Training procedure:
1. Pretraining: Freeze LLM and Encoder. Only train the projection layer/s for Vision-Language alignment. 
2. Instruction and alignment tuning: Train projection layer and LLM for multimodal tasks.

Examples of models in this category: LLaVA, PaLM-E, MiniGPT-v2, BLIP-2, MiniGPT-4, Video-LLaMA, defics2, MM1

### Type D: Tokenized Early Fusion (TEF)
![Type D](/posts/20241101_the_evolution_of_multimodal_model_architectures/type_d.png)

Models of this type are trained auto-regressively to generate text tokens along with other modalities tokens (image, audio, and videos).

1. Uses a pretrained LLM as the decoder
2. Modality-specific encoders are applied
3. Tokenizes all modalities at the input (discrete tokens)
4. Capable of generating outputs in multiple modalities

Examples of models in this category: LaVIT, TEAL, CM3Leon, VL-GPT, Unicode, SEED, 4M, Unified-IO, Unified-IO-2.

## Resource
[arxiv](https://arxiv.org/pdf/2405.17927)