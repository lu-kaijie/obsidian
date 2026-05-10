---
tags: [摘要, ai, anthropic, evals, coding, swe-bench]
sources: [raw/ai/anthropic/engineering/2026-05-10-claude-swe-bench-performance-webclip.md]
updated: 2026-05-10
---

# Claude SWE-Bench Performance

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/swe-bench-sonnet)

## 核心结论

这篇文章最值得保留的，不是单纯 benchmark 分数，而是它说明 `SWE-Bench 成绩依赖 agent + tool use + harness`，而不是单看模型裸能力。

## 为什么值得看

- 适合作为 Anthropic 在 coding eval 上的代表页。
- 与 [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]、[[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]] 可以形成一条完整评测主线。
- 用例层面也能帮助理解 Claude 在真实代码修复任务中的行为模式。

## 阅读评级

🟡 值得追问 — 如果你需要 Claude coding 能力的官方评测样本，这篇值得留存。

## 与其他页面的关联

- [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]
- [[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|Quantifying infrastructure noise in agentic coding evals]]
