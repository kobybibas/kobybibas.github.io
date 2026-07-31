---
title: "[Summary] Why AI Systems Don't Learn (and What to Do About It)"
date: 2026-07-31
tags:
  - Autonomous Learning
  - Reinforcement Learning
  - Self-Supervised Learning
  - Cognitive Science
draft: false
---

**TL;DR** Current AI systems do not learn autonomously after deployment. The usual process includes fine-tuning, in which a human carefully selects data for a specific task and tunes the model on it. This paper proposes combining learning from observation (System A), learning from action (System B), and a meta-controller (System M) that decides what and how to learn.

## Why Current AI Systems Do Not Learn

The current AI paradigm focuses on scaling LLMs. However, this approach exhausts the available high-quality data, focuses on language rather than embodied reasoning, and cannot discover new knowledge through interaction or learn continually after deployment.

Today, humans collect the data, define objectives, and decide the training stages. Once deployed, the model usually stops learning. The paper argues that learning should instead be an intrinsic capability built from three systems:

- **System A:** learning from observation, primarily through self-supervised and predictive learning.
- **System B:** learning from actions, primarily through reinforcement learning.
- **System M:** switching between learning modes using internally generated meta-control signals.

## System A and System B Relationship

System A learns representations and predictive world models from observations, but it cannot decide which data to acquire next. System B learns through interaction using external rewards or internal signals such as curiosity and novelty, but reinforcement learning is often sample-inefficient.

System A improves System B in three ways:

1. It learns compressed representations of states and actions.
2. It supplies predictive world models so System B can plan instead of blindly relying on trial and error.
3. It generates intrinsic rewards such as prediction error and uncertainty, guiding exploration toward where knowledge is thin.

System B helps System A in two ways:

1. In **active self-supervised learning**, System B's movements are steered by System A's uncertainty to select interesting data.
2. In **goal-directed self-supervised learning**, System B pursues its task and, as a byproduct, produces data for System A.

![Systems A and B interaction](/posts/20260731_why_ai_systems_dont_learn/system_a_b_interactions.png)

## System M: Meta-Control for Autonomous Learning

Currently, interactions between Systems A and B are managed offline by humans. An LLM, for example, goes through disconnected stages of next-token prediction and RLHF. System M instead monitors meta-states such as prediction error and uncertainty, then controls information flow between Systems A, B, and episodic memory.

System M has three main responsibilities:

1. **Input selection:** decide which observations or memories are useful.
2. **Loss and reward modulation:** adjust objectives using novelty.
3. **Mode-of-operation control:** decide when to learn, plan, act, or replay memories.

This routing also enables learning from trusted communication and imagined experience.

**Bootstrapping the Architecture.** An important and debatable choice is that System M's core policy is hardwired during development rather than learned alongside A and B.

A needs action-generated data, B needs representations from A, and M needs calibrated signals from both. The paper addresses this bootstrapping problem with bilevel optimization: A and B learn during an agent's lifetime under a fixed M, while an outer evolutionary loop optimizes the initial configuration of all three systems across simulated lifetimes.

## Limitations (per the paper)

This is a conceptual blueprint, not an evaluated system. Its components exist separately in active learning, world models, intrinsic motivation, and inference-time compute, but the paper does not demonstrate a unified architecture. Building one requires realistic fast simulators and expensive optimization across many lifetimes.

## References

- [Why AI systems don't learn and what to do about it](https://arxiv.org/abs/2603.15381), Emmanuel Dupoux, Yann LeCun, and Jitendra Malik, 2026.
