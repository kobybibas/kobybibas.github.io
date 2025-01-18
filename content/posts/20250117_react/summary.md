---
title: "[Summary] ReAct: Synergizing Reasoning and Acting in Language Models
date: 2025-01-17
tags: 
- Large Language Models 
- Prompt engineering
draft: true 
---

## TL;DR 

Tight synergy between “acting” and “reasoning” allows humans to learn new tasks quickly and perform robust decision making or reasoning, even under previously unseen circumstances or facing information uncertainties.

However, this “chain-of-thought” reasoning is a static black box, in that the model uses its own internal representations to generate thoughts and is not grounded in the external world, which limits its ability to reason reactively or update its knowledge. 

There have not been studies on how reasoning and acting can be combined in a synergistic manner for general task solving, and if such a combination can bring systematic benefits compared to reasoning or acting alone.

## ReAct
ReAct, a general paradigm to combine reasoning and acting with language
models for solving diverse language reasoning and decision making tasks 

ReAct prompts LLMs to generate both verbal reasoning traces and actions pertaining to a task in an
interleaved manner, which allows the model to perform dynamic reasoning to create, maintain, and
adjust high-level plans for acting (reason to act), while also interact with the external environments
(e.g. Wikipedia) to incorporate additional information into reasoning (act to reason).

there could be various types of useful thoughts, e.g. decomposing task goals and create action plans
(2b, Act 1; 1d, Thought 1), injecting commonsense knowledge relevant to task solving (2b, Act 1),
extracting important parts from observations (1d, Thought2, 4), track progress and transit action plans
(2b, Act 8), handle exceptions and adjust action plans (1d, Thought 3), and so on.

The setup is a frozen large language model, PaLM-540B (Chowdhery et al., 2022)1, is prompted with few-shot in-context examples to generate both domain-specific actions and free-form language thoughts for task solving (Figure 1 (1d), (2b)). Each in-context example is a human trajectory of actions, thoughts, and environment observations to solve a task instance 

![Prompt examples](/posts/20250117_react/react_method.png)

## Limitation 
we analyze the limitations of ReAct under the
prompting setup (i.e. limited support of reasoning and acting behaviors), and perform initial finetuning
experiments showing the potential of ReAct to improve with additional training data. Scaling up
ReAct to train and operate on more tasks and combining it with complementary paradigms like
reinforcement learning could further unlock the potential of large language models.





## Resource
[arxiv](https://arxiv.org/pdf/2210.03629)
