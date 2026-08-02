---
layout: post
title: "Call stack diffs"
date: 2026-08-02 10:00:00 +0300
categories: engineering
---

Inspired by [HumanLayer's "Program Design" section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md#program-design) in *Why Software Factories Fail*, I've recently added a permanent system instruction for coding agents to communicate all planned changes or implementations inline as *call stack diffs*:

```diff
 LLMRun.generate()
 └─ llmClient.streamChatCompletions()
-   └─ iterator.next()
-      └─ wait indefinitely
+   └─ withProviderInactivityTimeout()
+      ├─ race iterator.next() against 30s timeout
+      ├─ event received → reset timer → yield downstream
+      └─ timeout
+         ├─ abort provider request
+         └─ throw ProviderInactivityTimeoutError
+            ↳ LLMRun.retryBackup()
```

I find this much faster to scan than the prose which these language models tend to produce by default. The red/green highlighting visually stands out from the rest of the session transcript, so you can quickly parse the behavioral changes on a high level. This is especially useful when working out the design in back-and-forth conversation or when resuming work after switching sessions.

This diff does not replace the supporting explanations or documentation, rather it's like a pseudocode but better at showing the location of changes in the actual program, using real function and class names in an informal but maximally compressed representation.
