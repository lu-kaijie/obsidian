---
tags: [摘要, ai, anthropic, sdk, agent, tools]
sources: [raw/ai/anthropic/engineering/2026-05-10-building-agents-with-the-claude-agent-sdk-webclip.md]
updated: 2026-05-10
---

# Building agents with the Claude Agent SDK

**来源：** [Claude Blog](https://claude.com/blog/building-agents-with-the-claude-agent-sdk)

## 核心结论

这篇文章最有价值的地方，是把“自己写 agent loop”与“用 SDK 组 agent”之间的边界讲清楚了。SDK 提供的不是魔法智能，而是更稳定的 `loop、上下文、工具和执行骨架`。

## 为什么值得看

- 它适合作为 Claude Agent SDK 的入门总览页。
- 与 [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]] 一起看，一篇偏方法论，一篇偏 SDK 落地。
- 对理解“给 Claude 一台电脑”和“Agentic search / 文件系统”这类能力模块很有帮助。

## 阅读评级

🟡 值得追问 — 如果你要自己搭基于 Claude 的 agent，这篇很适合作为起点。

## 与其他页面的关联

- [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]]
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]
