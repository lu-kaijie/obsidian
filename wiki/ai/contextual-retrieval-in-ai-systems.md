---
tags: [摘要, ai, anthropic, rag, retrieval, context]
sources: [raw/ai/anthropic/engineering/2026-05-10-contextual-retrieval-in-ai-systems-webclip.md]
updated: 2026-05-10
---

# Contextual Retrieval in AI Systems

**来源：** [Anthropic Engineering](https://www.anthropic.com/engineering/contextual-retrieval)

## 核心结论

这篇文章的价值在于把传统 RAG 的“只切块不补上下文”问题讲清楚，并提出 `Contextual Retrieval`：在索引和检索时给 chunk 补上下文，让检索质量更接近真实语义边界。

## 为什么值得看

- 它适合作为 RAG / retrieval 主线中的 Anthropic 官方代表作。
- 与 [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] 可以互相补强：那篇偏架构组合，这篇偏 retrieval 细节。
- 也和 Prompt Caching / context efficiency 有直接关系，不只是搜索问题。

## 阅读评级

🔴 建议深读 — 如果你在做知识检索，这篇属于很值得保存的一手工程方法文章。

## 与其他页面的关联

- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]]
- [[wiki/ai/effective-context-engineering-for-ai-agents|Effective context engineering for AI agents]]
