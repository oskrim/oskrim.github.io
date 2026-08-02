---
layout: post
title: "Call stack diffs: a compact map for code review"
date: 2026-08-02 10:00:00 +0300
categories: engineering
---

A code diff tells us which lines changed. A call stack diff adds a small map of
how control flow changed at runtime.

For example, a retry might be summarized like this:

```diff
- request -> handler -> provider -> error
+ request -> handler -> recovery boundary -> provider
+ recovery boundary -> classify failure -> retry or surface error
```

This is useful in reviews because it makes the behavioral change visible
without requiring every reader to reconstruct it from several files. It also
gives reviewers a quick way to check important branches: success, retries,
fallbacks, state transitions, and failures.

The format does not replace tests or code. It is simply a concise companion to
them—especially handy when a small patch changes where responsibility lives or
introduces a new path through the system.

This is a placeholder first post. I plan to add concrete examples from real
changes later.
