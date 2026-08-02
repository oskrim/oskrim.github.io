---
layout: post
title: "Call stack diffs"
date: 2026-08-02
categories: engineering
---

Before writing the code, sketch the call stack as pseudocode. Sketch the *change* as a diff. I picked the habit up from the [Program Design section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md#program-design) of HumanLayer's *Why Software Factories Fail*.

Here's a plan for taking order processing async:

```diff
  POST /orders
    validate(order)
-   charge(order)
-   send_confirmation(order)
-   return 201
+   enqueue(order)
+   return 202

+ worker
+   charge(order)
+     on decline: cancel + notify
+   send_confirmation(order)
```

The whole design is legible from the shape. The hot path shrinks to an enqueue. `201` → `202` is the API contract changing, one character that would drown in a real PR. `validate` staying put shows exactly where the sync/async boundary landed. And the one hard question (what happens to a declined card once the caller is gone) can't hide: it's the green branch.

The same works for file trees, type signatures, and pseudocode: print them all as diffs against the previous version. Put a permanent instruction into your `AGENTS.md` to include a call stack diff with every plan and every implementation summary, you won't regret it.
