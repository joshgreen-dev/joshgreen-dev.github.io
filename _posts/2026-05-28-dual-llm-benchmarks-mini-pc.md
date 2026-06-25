---
layout: post
title: "I Ran Two Local LLMs on a Mini PC at Once. Benchmarks Show Why It's Pointless."
date: 2026-05-28
tags: [ai, llm, minipc, selfhosted]
description: "Dual-model benchmarks on a 96GB UM790Pro. The shared DDR5 bus ruins everything, but MoE saves the day."
---

If you've been following my stuff, you know I'm all about squeezing maximum value out of minimal hardware. Mini PCs, home labs, self-hosted everything. So naturally, when I got my hands on a UM790Pro with 96 GB of DDR5, my first thought was: can I run two LLMs simultaneously?

The answer is yes. The better question is: should you?

No. I have the benchmarks.

## The Setup

The UM790Pro is a beast for its size. Here's what I'm running on:

- **CPU:** AMD Ryzen 9 7940HS
- **GPU:** AMD 780M iGPU (integrated, shares system memory)
- **RAM:** 96 GB DDR5-5600
- **VRAM Pool:** 2 GB dedicated + 46 GB GTT = 48 GB total GPU-accessible memory
- **Memory Bandwidth:** ~80 GB/s (shared between CPU and iGPU)

That last point is the key to everything that follows. On a discrete GPU, the CPU and GPU have their own separate memory buses. On an APU like the 7940HS, the CPU and iGPU drink from the same straw. DDR5-5600 gives you roughly 80 GB/s, and both the CPU cores and the GPU compute units fight over every byte of it.

I'm running Ollama as my inference server. Four models in the ring:

| Model | Params | Size on Disk | Max Context | Type |
|---|---|---|---|---|
| qwen3.6:35b | 36B total (MoE, 256 experts, 8 active) | 23.9 GB | 262K | Mixture of Experts |
| gemma4-e2b-abliterated | 4.6B | 3.4 GB | 131K | Dense |
| qwen3:4b-instruct | 4B | 2.5 GB | 256K | Dense |
| qwen2.5:1.5b-cpu | 1.5B | 1.0 GB | 32K | Dense |

The 35B MoE model is the big gun, my daily driver for coding and complex reasoning. The smaller models were candidates for a "sidecar" role: handling quick tasks like summarization or classification while the big model crunches harder problems.

## Baseline: One Model at a Time

First, I benchmarked each model running alone to get clean numbers.

| Model | GPU (tok/s) | CPU (tok/s) |
|---|---|---|
| qwen3.6:35b | 17.8 | N/A |
| gemma4-e2b-abliterated | 42.9 | 28.7 |
| qwen3:4b-instruct | 26.2 | 19.6 |
| qwen2.5:1.5b-cpu | N/A | 53.4 |

The 35B model at 17.8 tok/s on an iGPU is genuinely impressive. That's usable for interactive chat. The small models are blazing fast. Gemma at 42.9 tok/s on GPU is practically instant for short responses.

Looking at these numbers, I thought: what if I keep the 35B on GPU and run a small model on CPU simultaneously? Best of both worlds, right?

## The Dual-Model Experiments

I ran four combinations, firing both models at the same time with identical prompts and measuring throughput.

### Test 1: Both Models on GPU

**qwen3.6:35b (GPU) + gemma4-e2b (GPU)**

| Model | Alone | Simultaneous | Drop |
|---|---|---|---|
| qwen3.6:35b | 17.8 tok/s | 13.1 tok/s | -26% |
| gemma4-e2b | 42.9 tok/s | 25.3 tok/s | -41% |

Both models fighting for the same GPU compute units and the same memory bus. The 35B model drops to 13.1 tok/s. Painful but expected.

### Test 2: Big Model GPU + Tiny Model CPU (The "Best" Result)

**qwen3.6:35b (GPU) + qwen2.5:1.5b (CPU)**

| Model | Alone | Simultaneous | Drop |
|---|---|---|---|
| qwen3.6:35b | 17.8 tok/s | 14.9 tok/s | -16% |
| qwen2.5:1.5b | 53.4 tok/s | 26.2 tok/s | -51% |

This was the best result. The 1.5B model is tiny enough that its CPU inference doesn't hammer memory bandwidth too hard. The big model only drops 16%. But the small model gets cut in half.

### Test 3: Big Model GPU + Medium Model CPU-Forced

**qwen3.6:35b (GPU) + gemma4-e2b (CPU, num_gpu=0)**

| Model | Alone | Simultaneous | Drop |
|---|---|---|---|
| qwen3.6:35b | 17.8 tok/s | 13.0 tok/s | -27% |
| gemma4-e2b | 28.7 tok/s | 13.4 tok/s | -53% |

Forcing Gemma to CPU didn't help. The 4.6B model doing CPU inference generates enough memory traffic to compete with the GPU's reads. Both models suffer.

### Test 4: The Worst Case (KV Cache Explosion)

**qwen3.6:35b (GPU) + qwen3:4b-instruct (CPU, num_gpu=0)**

| Model | Alone | Simultaneous | Drop |
|---|---|---|---|
| qwen3.6:35b | 17.8 tok/s | 11.6 tok/s | -35% |
| qwen3:4b-instruct | 19.6 tok/s | 11.1 tok/s | -43% |

This was the disaster scenario. The 4B instruct model supports a 256K context window, and its KV cache ballooned to 24.2 GB at full context. Combined with the 35B model's 32 GB VRAM allocation, we were pushing close to the system's total memory bandwidth capacity. Both models crawled.

## The Memory Architecture Problem

What's actually happening inside this machine:

```
+--------------------------------------------------+
|                  DDR5-5600 (96 GB)                |
|            ~80 GB/s shared bandwidth              |
+--------------------------------------------------+
        |                           |
   +---------+              +-----------+
   | CPU     |              | 780M iGPU |
   | Cores   |              | 12 CUs    |
   | (Zen 4) |              |           |
   +---------+              +-----------+
        |                        |
   CPU inference          GPU inference
   (model weights          (model weights
    in system RAM)          in VRAM/GTT)
        |                        |
        +------- SAME BUS ------+
```

The VRAM pool breaks down like this:

- **2 GB dedicated VRAM:** physically reserved for the iGPU
- **46 GB GTT (Graphics Translation Table):** system RAM mapped into GPU address space
- **48 GB total GPU-accessible memory**

When both a GPU model and a CPU model are running, they're both streaming weights from the same DDR5 DIMMs through the same memory controller. The GPU doesn't have its own GDDR6 with 300+ GB/s bandwidth like a discrete card. It's sharing the same 80 GB/s pipe as everything else.

It's not a compute bottleneck. It's a memory bandwidth bottleneck.

## The Real-World Conclusion

I was testing this because I wanted to run an agent framework: a planning model plus an execution model working together. The idea was the big 35B model handles complex reasoning while a small model handles quick tool-calling or classification.

But agent frameworks run tasks sequentially, not in parallel.

The planner thinks, then the executor acts, then the planner thinks again. They're taking turns. At any given moment only one model is generating tokens. The other is just sitting there, loaded in memory, doing nothing but occupying VRAM or RAM that could go toward bigger context windows instead.

So the dual-model setup gives you:

1. **Worse throughput** on the big model (11-15 tok/s vs 17.8 tok/s)
2. **No parallelism benefit** in sequential agent workflows
3. **Wasted memory** keeping a second model loaded
4. **Risk of OOM crashes** (Ollama's iGPU memory reporting has a known bug, [issue #14953](https://github.com/ollama/ollama/issues/14953), that can cause crashes with multiple loaded models)

## The MoE Insight

Here's the moment that made me feel silly for even running these tests.

The qwen3.6:35b model is a Mixture of Experts architecture. It has 256 experts but only activates 8 per token. For any given token, it's doing roughly the compute of a 4-5B parameter model while having the knowledge of a 36B parameter model.

Read that again. The big model already IS the small model in terms of per-token compute cost. MoE gives you the reasoning depth of 35B parameters with the inference speed of a much smaller model. Running a separate small model alongside it for "fast tasks" is solving a problem that doesn't exist.

17.8 tok/s for 35B-class reasoning is already fast enough for everything I throw at it. Adding a second model only makes it slower.

## Bonus: Ollama Storage Gotchas

While poking around, I found a couple things worth mentioning.

**Shared blobs save disk space.** I had `qwen3.6:35b`, `qwen3.6:latest`, and `qwen3.6:35b-nothink` all listed as separate models. Turns out they all point to the same 23.9 GB blob on disk. Ollama uses content-addressed storage, so identical weights are stored once regardless of how many tags reference them.

**Orphan blobs waste disk space.** After deleting some models, I found a 12.9 GB orphan blob sitting in `~/.ollama/models/blobs/` that no tag referenced anymore. There's no `ollama prune` command yet, so I had to manually cross-reference blob hashes against manifest files and delete the orphan by hand. Check yours. You might be surprised.

## TL;DR

Running two LLMs simultaneously on a shared-memory APU is technically possible but practically pointless. DDR5 bandwidth (~80 GB/s) is the bottleneck, not compute. Both models compete for the same memory bus regardless of CPU vs GPU assignment. Agent frameworks run sequentially anyway, so there's no parallel benefit. MoE models like qwen3.6:35b already give you big-model reasoning at small-model speeds.

Just run one model. Use the freed memory for bigger context windows. That's where shared-memory APUs actually shine.

---

*Tested on: Minisforum UM790Pro, AMD Ryzen 9 7940HS, 96 GB DDR5-5600, Ollama v0.9.x, Ubuntu Linux*
