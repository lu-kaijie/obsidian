---
tags: [摘要, ai, anthropic, mcp, tools, efficiency]
sources: [raw/ai/anthropic/engineering/2026-05-10-code-execution-with-mcp-building-more-efficient-ai-agents-webclip.md]
updated: 2026-05-10
---

# Code execution with MCP: building more efficient AI agents

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/code-execution-with-mcp)

## 核心结论

这篇文章最重要的判断是：工具不仅影响能力，还影响 `上下文成本`。如果工具定义过大、工具结果过长，agent 的 token 会被工具噪声吃掉。MCP 里的代码执行能力，本质上是在做 `更高上下文效率的工具设计`。

## 为什么值得看

- 它把 “工具效率” 和 “上下文工程” 直接连起来了。
- 与 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]、[[wiki/ai/writing-effective-tools-for-ai-agents-using-ai-agents|Writing effective tools for AI agents—using AI agents]] 强相关。
- 适合作为 “progressive disclosure” 在工具层的具体案例页。

## 阅读评级

🔴 建议深读 — 如果你在做 agent tools，这篇对“怎么让工具不把上下文拖死”非常关键。

## 与其他页面的关联

- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]
- [[wiki/ai/writing-effective-tools-for-ai-agents-using-ai-agents|Writing effective tools for AI agents—using AI agents]]
- [[wiki/ai/effective-context-engineering-for-ai-agents|Effective context engineering for AI agents]]
