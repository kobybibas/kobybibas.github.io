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
Direct Preference Optimization is a method of fine-tuning Large Language Models (LLM) to better align their outputs with human preference.
It's used as a simpler alternative to RLHF since it can be directly applied to the model without needing a reward function nor reinforcement learning optimization. 

## Method
The authors propose to re-parameterize the reward model of RLHF to obtain the optimal policy in closed form.
This enables to solve the standard RLHF problem using a simple classification loss. 

To align with human preference, DPO requires the following steps:
1. Train an LLM with unsupervised data.
2. Given a prompt, feed it twice to the LLM to generate a pair of responses. Annotate one as positive and the other as negative based on "human preference".
3. Train the LLM directly with the dataset of (2) with the following loss function:
![DPO loss function](/posts/20231223_dpo/dpo_loss_function.png)
where y_w and y_l are the positive and negative samples. The denominator in loss function (\phi_ref), keeps the model to not diverge too much from the original model weights.

![DPO update step](/posts/20231223_dpo/dpo_update_step.png)

\
The following figure (taken from AI Coffee Break with Letitia [2]) illustrates the difference between RLHF and DPO.
![RLHF vs DPO](/posts/20231223_dpo/rlhf_vs_dpo.png)


## A side note
Can this loss function be applied to a model that uses the triplet loss?  It might provide better performance since it's the closed form of the optimal policy.

## Resource

[1] [DPO paper](https://arxiv.org/abs/2305.18290)

[2] [AI Coffee Break with Letitia: Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://www.youtube.com/watch?v=XZLc09hkMwA)


