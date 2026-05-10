---
tags: [摘要, ai, anthropic, tools, reasoning, claude]
sources: [raw/ai/anthropic/engineering/2026-05-10-the-think-tool-enabling-claude-to-stop-and-think-webclip.md]
updated: 2026-05-10
---

# The "think" tool: Enabling Claude to stop and think

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/claude-think-tool)

## 核心结论

这篇文章的关键，不是“让模型多想一会儿”这么简单，而是把 `显式思考停顿` 做成可控工具。它说明 reasoning 有时不是 prompt 里喊一声“think step by step”，而是需要一个运行时结构。

## 为什么值得看

- 它适合作为 tool-augmented reasoning 的官方样本。
- 和 [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]]、[[wiki/ai/introducing-advanced-tool-use-on-the-claude-developer-platform|Introducing advanced tool use on the Claude Developer Platform]] 能构成很好的工具层主线。
- 也能帮助理解为什么某些 benchmark 上显式 think tool 会有效。

## 阅读评级

🟡 值得追问 — 如果你关心 agent 推理结构如何外显成工具，这篇值得看。

## 与其他页面的关联

- [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]]
- [[wiki/ai/introducing-advanced-tool-use-on-the-claude-developer-platform|Introducing advanced tool use on the Claude Developer Platform]]
