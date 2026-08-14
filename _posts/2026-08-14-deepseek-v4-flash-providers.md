---
layout: post
title: "Deepseek V4 Flash 0731 latency numbers from nine providers"
date: 2026-08-14
categories: engineering
---

DeepSeek V4 Flash 0731 is available from many inference providers. I wanted to
see which ones could handle production traffic.

I sent 30 streaming requests concurrently to each provider across four
workloads: 120 per provider and 1,080 total. Decode speed is the combined rate
of the 30 decode requests.

This is a one-time snapshot. DeepInfra felt faster earlier, before US traffic
ramped up.

## Results

| Provider | Decode tok/s | TTFT p50 / p99 | Total p99 | Cache hit | Errors |
|:--|--:|--:|--:|--:|--:|
| Scaleway | 1,650 | 733 / 818 ms | 18.62 s | 80.2% | 18/120 |
| TensorX | 1,192 | 873 / 1,630 ms | 25.77 s | 73.4% | 1/120 |
| Fireworks | 2,333 | 1,072 / 2,592 ms | 13.17 s | 17.6% | 0/120 |
| Baseten | **3,980** | 774 / 2,979 ms | **7.72 s** | 78.7% | 0/120 |
| Azure | 596 | **618** / 3,029 ms | 51.55 s | 57.7% | 0/120 |
| DeepInfra | 685 | 1,466 / 4,663 ms | 44.85 s | 86.6% | 0/120 |
| DigitalOcean | 1,280 | 985 / 7,412 ms | 24.00 s | 26.5% | 0/120 |
| Nebius | 2,542 | 948 / 7,563 ms | 12.08 s | 0.0% | 0/120 |
| Lyceum | 934 | 8,275 / 9,354 ms | 32.89 s | **96.0%** | 0/120 |

Azure wasn't running the new DeepSeek V4 Flash 0731 checkpoint, so its results
aren't directly comparable with the others.

Scaleway's 18 failures were HTTP 429s from a token-per-minute quota; successful
requests had the best TTFT distribution. TensorX had one transient HTTP 502.
Everything else succeeded.
