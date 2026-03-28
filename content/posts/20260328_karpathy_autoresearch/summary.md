---
title: "[Summary] AutoResearch: Hard Constraints Enable Reliable Agentic ML"
date: 2026-03-28
tags:
  - Large Language Models
  - Agents
  - Training
  - Automation
draft: false
---

**TL;DR** AI coding agents typically produce code, and a human reviewer verifies it or tests catch regressions. AutoResearch is a different paradigm: an agent iterates on an ML model to improve a continuous metric. Its success comes from constraining the agent to a fixed time budget and defining an evaluation that is invariant to the code changes it is allowed to make. The implications go beyond ML models: the same approach could apply to latency, capacity, or any system with a measurable objective.

## Motivation

The bottleneck in ML research isn't ideas — it's the cost of running experiments. Each iteration requires implementation, a full training run, and manual evaluation. AutoResearch addresses this by making experiments cheap, time-bounded, and automatically reversible, turning the iteration loop into something an agent can run overnight.

## Method

AutoResearch succeeds because of a tightly constrained setup with one specific job:
1. Let an agent modify a training script
2. Run a bounded experiment (by time)
3. Measure the result using a fixed metric
4. Keep the change if it helps, discard it if it does not

Out of scope for the agent: browsing papers or coming up with novel theories.

Karpathy's repository contains 3 files only:
1. `prepare.py` — fixed harness: dataset, tokenizer, dataloader, evaluation. Agent cannot touch this.
2. `train.py` — the only file the agent is allowed to change.
3. `Program.md` — instructions for the agent.

What makes the constraints work:
1. **Metric:** eval uses bits per byte (`val_bpb`) rather than raw loss, making results tokenizer-invariant.
2. **Simplicity bias:** a small improvement with added complexity is not worth it; equal performance with less code is a win.
3. **Frontier tracking:** all experiments are logged to `results.tsv`, which serves as the agent's file-based memory:

```
commit   val_bpb  memory_gb  status   description
a1b2c3d  0.9979   44.0       keep     baseline
b2c3d4e  0.9932   44.2       keep     increase LR to 0.04
c3d4e5f  1.0050   44.0       discard  switch to GeLU activation
```

4. **Single long-running session:** each iteration starts from the current best commit. Output is redirected to `run.log` to avoid flooding the context window.
5. **Time-bounded experiments:** ~5 min target, killed at 10 min and treated as failure.
6. **Never stop:** the agent is explicitly instructed not to pause for human confirmation once the loop starts.

## Limitations

1. Single continuous session: context degrades over many iterations.
2. Finds local optima, not novel approaches — creativity is fully out of scope.
3. Metric quality is still a human judgment call; a bad metric means wasted compute.
4. Only works when experiments are fast and cheap to run.

## Related Approaches

**[Evolver](https://github.com/eranshir/evolver)** — a more sophisticated take: the user can propose hypotheses, and multiple agents run experiments in parallel. More powerful, but less hands-off.

**[Ralph Wiggum](https://github.com/anthropics/claude-code/blob/main/plugins/ralph-wiggum/README.md)** — similar loop paradigm but with binary criteria: the agent iterates until a fixed test passes rather than optimizing a continuous metric. Same principle, different termination logic.

**[Claude Autoresearch](https://github.com/uditgoenka/autoresearch)** — a Claude Code skill that generalizes AutoResearch beyond ML to any domain with a measurable metric. Adds a Guard mechanism: a secondary command that must pass to prevent regressions while the primary metric improves.

## References

- [karpathy/autoresearch](https://github.com/karpathy/autoresearch/tree/master)
- [AutoResearch thread](https://x.com/manthanguptaa/status/2032464949952598152?s=20)
