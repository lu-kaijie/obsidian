---
tags: [摘要, ai, anthropic, evals, infra, noise]
sources: [raw/ai/anthropic/engineering/2026-05-10-quantifying-infrastructure-noise-in-agentic-coding-evals-webclip.md]
updated: 2026-05-10
---

# Quantifying infrastructure noise in agentic coding evals

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/infrastructure-noise)

## 核心结论

这篇文章最关键的提醒是：agentic coding eval 的波动，很多时候不是 agent 变聪明或变笨，而是 `基础设施噪声` 在作祟。也就是说，评测误差有时来自环境，不来自模型。

## 为什么值得看

- 它能校正很多人对 benchmark 波动的直觉。
- 和 [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]、[[wiki/ai/a-postmortem-of-three-recent-issues|A postmortem of three recent issues]] 可以连成一条稳定性主线。

## 阅读评级

🔴 建议深读 — 如果你在跑 coding eval，这篇很容易帮你少踩很多“误读分数”的坑。

## 与其他页面的关联

- [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]
- [[wiki/ai/a-postmortem-of-three-recent-issues|A postmortem of three recent issues]]
