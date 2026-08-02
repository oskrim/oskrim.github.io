---
layout: post
title: "Call stack diffs"
date: 2026-08-02 10:00:00 +0300
categories: engineering
---

Inspired by the idea from [HumanLayer's "Program Design"
section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md#program-design)
in *Why Software Factories Fail*, I've recently added a permanent system instruction for all coding agents to communicate all planned changes or implementations inline as *call stack diffs*:

```diff
 LLMRun.generate()
 └─ llmClient.streamChatCompletions()
-   └─ iterator.next()
-      └─ wait indefinitely
+   └─ withProviderInactivityTimeout()
+      ├─ race iterator.next() against 30s timeout
+      ├─ event received
+      │  ├─ clear timer
+      │  └─ yield event downstream
+      ├─ timeout
+      │  ├─ abort provider request
+      │  ├─ throw ProviderInactivityTimeoutError
+      │  └─ LLMRun.retryBackup()
```

This feels a lot more efficient to parse than wading through the typical prose that is produced by these language models by default. The changed and new code paths are easy to scan. This feels especially useful in back-and-forth conversation and and when context switching between sessions.

This is much easier to parse than wading through the prose language models produce by default. Changed and new code paths are immediately scannable, which is especially useful in back-and-forth conversations and when switching between sessions.
