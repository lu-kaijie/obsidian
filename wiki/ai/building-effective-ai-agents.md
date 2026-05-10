---
tags: [摘要, ai, anthropic, agent, architecture, workflow]
sources: [raw/ai/anthropic/engineering/2026-05-10-building-effective-ai-agents-webclip.md]
updated: 2026-05-10
---

# Building Effective AI Agents

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/building-effective-agents)

## 核心结论

这篇几乎可以看作 Anthropic 对 agent engineering 的总论页。最关键的判断是：`不要过度神化 agent`，先从 augmented LLM 和 workflow 组件出发，再按任务需要逐步提高自主性。

## 为什么值得看

- 它把 prompt chaining、routing、parallelization、orchestrator-workers 等模式讲成了一套清晰的积木体系。
- 很适合作为现有知识库多篇 agent 架构文章的“第一性原理入口”。
- 与 [[wiki/ai/agent-principles-architecture-engineering-practice|你不知道的 Agent]] 是强互补：一个来自 Anthropic，一篇是本地总结。

## 阅读评级

🔴 建议深读 — 如果你要给别人解释“agent 到底是什么、什么时候真该上 agent”，这篇是非常强的一手材料。

## 与其他页面的关联

- [[wiki/ai/agent-principles-architecture-engineering-practice|你不知道的 Agent]]
- [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]]
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]
