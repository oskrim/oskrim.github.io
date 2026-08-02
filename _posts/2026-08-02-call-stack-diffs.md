---
layout: post
title: "Call stack diffs"
date: 2026-08-02 10:00:00 +0300
categories: engineering
---

I picked up this idea from [HumanLayer's "Program Design"
section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md#program-design)
in *Why Software Factories Fail*.

Architecture describes how the larger pieces fit together. Program design goes
one level deeper, shaping the types, method signatures, file layout, and call
stacks before implementation begins. A call stack diff is a lightweight
pseudocode view of that design: it shows how orchestration and control flow are
expected to change.

What clicked for me is that this gives you a compressed, visual representation
of a plan or implementation. Instead of holding the whole code path in your
head, you can review its shape directly.

For example, a retry might be summarized like this:

```diff
- request -> handler -> provider -> error
+ request -> handler -> recovery boundary -> provider
+ recovery boundary -> classify failure -> retry or surface error
```

Rendered with the usual red and green diff colors, the old and new paths are
surprisingly easy to scan. The coloring is not just cosmetic: it makes a new
branch, moved responsibility, or removed fallback stand out immediately.

This is useful during planning because important decisions become explicit
before they are buried in code. It is useful during review because the intended
behavior can be compared with the implementation without reconstructing the
whole path from several files.

Call stack trees are also part of a broader program-design toolkit: file-tree
diffs show where code will live, type signatures pin down boundaries, and
pseudocode diffs make algorithm changes reviewable. Most of these can be shown
as a diff against the previous design, which is often the most useful form.

A practical way to make this routine is to add a standing instruction to your
`AGENTS.md`: ask the agent for call-stack diffs during planning and again after
implementation. The format does not replace tests or code, but it is a compact
companion to both.
