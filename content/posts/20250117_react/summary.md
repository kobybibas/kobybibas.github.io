---
title: "[Summary] ReAct: Synergizing Reasoning and Acting in Language Models"
date: 2025-01-17
tags:
  - Large Language Models
  - Prompt Engineering
  - Chain of Thought
  - Reasoning
draft: false 
---

## TL;DR 
Large Language Models (LLMs) often suffer from hallucinations. Two common mitigation strategies are Chain of Thought (CoT), where the LLM is prompted to show its step-by-step reasoning, and Act, where LLMs use external tools to ground their answers in reliable databases. However, CoT relies on the model's internal representations, limiting its ability to reason reactively or update its knowledge. 
ReAct is a prompting method that combines CoT with action plan generation using external tools. This approach significantly enhances LLM performance in decision-making and question answering.

## ReAct
ReAct involves prompting an LLM with few-shot in-context examples to generate both domain-specific actions and free-form language thoughts for task solving. Each example includes a human trajectory of actions, thoughts, and environment observations:
- Thought: ...
- Action: ...
- Observation: ...

In experiments, the following useful thoughts were observed:
1. Decomposing task goals and creating action plans.
2. Injecting commonsense knowledge relevant to task solving.
3. Extracting important parts from observations.
4. Tracking progress and transitioning action plans.
5. Handling exceptions and adjusting action plans.

![Prompt examples](/posts/20250117_react/react_method.png)

## Limitations
1. Dependency on in-context examples, which may not always be available
2. Increased computational cost due to the need for generating both thoughts and actions
3. Reliance on external tools, which may introduce latency or require integration efforts.

## Resources
[arxiv](https://arxiv.org/pdf/2210.03629)
