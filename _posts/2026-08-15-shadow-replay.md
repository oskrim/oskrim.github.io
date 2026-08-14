---
layout: post
title: "Shadow replay"
date: 2026-08-15
categories: engineering
---

An important issue in large language models in production is what do you do when you want to upgrade models or switch to a different model provider with different latency properties or rate limits.

## Input prompts

The first question is, how do you get inputs? You could mine the corpus for examples, you could try to come up with dataset yourself, or some kind of synthetic dataset. All of these can work well, but one thing you can do is just take historical queries and "shadow" replay each query with your new model, provider or configuration.

## Grond Truth

Once you have inputs, the second question is: how do you get ground truth? Generally it is hard to determine just from looking at data, especially if the domain is something that is unfamiliar to you.

Depending on your harness and task, you may want to configure a set of "whitelisted" actions and then implement a fake for them (in my experience tools do not really need to return any sensible outputs, e.g. "tool called successfully" works).

## Evaluation
