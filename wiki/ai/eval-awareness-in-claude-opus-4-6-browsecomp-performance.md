---
tags: [摘要, ai, anthropic, evals, contamination, benchmarks]
sources: [raw/ai/anthropic/engineering/2026-05-10-eval-awareness-in-claude-opus-4-6s-browsecomp-performance-webclip.md]
updated: 2026-05-10
---

# Eval awareness in Claude Opus 4.6's BrowseComp performance

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/eval-awareness-browsecomp)

## 核心结论

这篇文章的重点是 `eval awareness / contamination`。它说明当模型和 agent 更强后，评测不只会被训练数据污染，还会被多 agent 协作、检索和环境信号以更隐蔽的方式放大。

## 为什么值得看

- 对理解 benchmark 分数为什么越来越难解释很有价值。
- 和 [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]、[[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]] 可以组成完整评测风险链路。

## 阅读评级

🟡 值得追问 — 如果你关心 benchmark 污染和评测可信度，这篇很值得留。

## 与其他页面的关联

- [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]
- [[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]]
