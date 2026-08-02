---
layout: post
title: "Call stack diffs"
date: 2026-08-02
categories: engineering
---

I'm sure many people already do this, but reviewing "compressed" or "visual" representations of the underlying plan or implementation has helped me a lot recently, both at the planning stage and at the review stage. Some of the specifics I picked up from the [Program Design section](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md#program-design) of HumanLayer's *Why Software Factories Fail*.

The framing there is that after architecture comes program design. Before anyone, human or agent, writes the implementation, you go one level down into the shape of the code: the types, the method signatures, the file layout, and the call stacks. A call stack diff is a pseudocode sketch of that last item, printed as a diff against the current version: how the control flow looks now, and how it's expected to change.

Here's one an agent printed for me recently while we were planning an eval-only reasoning-effort override for our chat API:

```diff
  realm-eval CLI
-   parse --model
+   parse --model + --reasoning-effort
    suite runner
      run_turn()
-       eval_options { include_turn_trace, model_override }
+       eval_options { include_turn_trace, model_override, reasoning_effort }
        POST /external/chat
          validate internal-eval access
+         validate reasoning_effort requires turn trace
          generateChat()
+           GenerationOptions.evalReasoningEffort
            AgentStep / AnswerStep
-             derive effort from thinking mode
+             use eval override, otherwise derive effort unchanged
              LLMClient → Azure Responses API
```

This looks very nice when colored red/green. A new branch, a moved responsibility, or a removed fallback stands out immediately, and at review time you can compare the intended shape against what actually got built without reconstructing the whole path from several files.

Call stacks are only one member of the toolkit. File tree diffs show where the code will live, type signatures pin down the boundaries, and pseudocode diffs make changes to algorithms reviewable. Usually all of these (callstacks/filetrees/types/pseudocode) can be printed out as diffs against the previous version, and I frequently find the diff to be the most useful format of them all.

To make it routine, put a permanent instruction into your `AGENTS.md` to print call stack diffs at every planning and implementation task, you won't regret it. It doesn't replace tests or reading the actual code, but as a companion to both it costs almost nothing.
