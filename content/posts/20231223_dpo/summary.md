---
title: "[Summary] Direct Preference Optimization (DPO)"
date: 2023-12-23
tags: 
    - Reinforcement learning
    - LLM
    - Reward function
draft: false
---

## TL;DR
Direct Preference Optimization is a method of fine-tuning Large Language Model (LLM) to better align with human preference. 
It's used as a simpler alternative to RLHF since it can be directly applied to the model without needing a reward function nor reinforcement learning optimization. 

## Method

The authors propose to re-parameterize the reward model in RLHF that enables extraction of the corresponding optimal policy in closed form.
This enables to solve the standard RLHF problem with only a simple classification loss. 

To align with human preference, DPO requires the following steps:
1. Train an LLM with unsupervised data
2. Given a prompts, feed it twice to the LLM and generate a pair of responses. Annotate positive and negative.
3. Train the LLM directly with the dataset of (2) with the following loss function:
![DPO loss function](/posts/20231223_dpo/dpo_loss_function.png)

The denominator in loss function, keeps the model to not diverge too much from the original model weights.

![DPO update step](/posts/20231223_dpo/dpo_update_step.png)

The following figure (taken from AI Coffee Break with Letitia [1]) illustrates the difference between RLHF and DPO.
![RLHF vs DPO](/posts/20231223_dpo/rlhf_vs_dpo.png)


## Resource

[AI Coffee Break with Letitia: Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://www.youtube.com/watch?v=XZLc09hkMwA)

[DPO paper](https://arxiv.org/abs/2305.18290)
