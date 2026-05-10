---
tags: [摘要, ai, coding, rag, spec, context]
sources: [raw/ai/2026-05-09-spec-rag-ai-programmer-knowledge-infra-webclip.md]
updated: 2026-05-09
---

# Spec + RAG 的 AI 程序员知识基础设施

**来源：** [翔旭, 2026-02-06](https://mp.weixin.qq.com/s/1NqfWn_kH5y-mBXAAu62aA)

## 核心结论

这篇文章最有价值的地方，是把“如何让 AI 真正懂项目”拆成了一套双层知识体系：

- `Spec` 负责项目级硬约束
- `RAG` 负责动态软上下文

再往后，它还顺势把 `MCP` 引入成外部能力接入层。这个三层组合，比单讲 `SDD` 或单讲 `RAG` 都更接近一个完整的 AI coding 知识基础设施方案。

## 它在解决什么问题

文章的起点很明确：当前很多 AI coding 工具的问题不是“不会生成代码”，而是“生成的代码并不真正理解项目的契约、语义和工程惯例”。

所以作者强调，想让 AI 写得对，光靠模型能力不够，关键要补它的“认知基础设施”。

它把这套基础设施分成两类：

- `Spec 知识库`：项目内部稳定、明确、可验证的规则和契约
- `RAG 知识库`：项目外部或半结构化的历史方案、最佳实践、文档与资料

这个拆法很有价值，因为它避免把所有上下文都混进一个检索库里。

## Spec 在这里被定义成“硬规则层”

文章对 `Spec` 的强调和现有 SDD 页面一致，但它有一个额外优点：把 `Spec` 和 `Vibe Coding` 的差异讲得非常直白。

核心区别可以压缩成：

- `Vibe Coding` 依赖意图模仿和主观理解
- `Spec Coding` 依赖书面规范和可验证契约

这其实就是把 `Spec` 从“开发前文档”重新定义成“生成代码时的硬约束集”。

和 [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] 的关系是：

- 那篇更像方法论总论
- 这篇更像在问：如果只靠 Spec 还不够，外部知识层应该怎么补

## RAG 在这里被定义成“软上下文层”

文章里对 `RAG` 的真正价值判断是：

- 不要求模型记住一切
- 而要求模型学会按需查找

这个视角并不新，但放到 AI coding 语境里很重要，因为它把项目知识分成了两类不同载荷：

- 必须被遵守的规范，不应该模糊检索
- 可以动态补充的背景知识，可以检索增强

也就是说：

- `Spec` 用来保证“不能错”
- `RAG` 用来帮助“更懂上下文”

这和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] 可以形成互补理解：skill 负责按层加载能力与 SOP，RAG 更偏按需召回背景材料与文档知识。

## 它把 RAG 讲成了工程问题，不只是检索技巧

文章里花了很大篇幅讲 chunking、embedding、query transformation、hybrid retrieval 和 reranking。虽然这些内容偏综述，但有一个好处：它提醒你 RAG 不是“接个向量库就结束”，而是一整套检索质量工程。

在 AI coding 场景里，这一点尤其重要，因为如果：

- chunk 切错
- query 改写不准
- rerank 不稳

那最终召回给模型的上下文就会失真，反而让“看起来更懂”的 AI 变成“更有底气地胡说”。

## MCP 在这篇里的位置也值得记

文章最后把 `MCP` 引入进来，不只是为了讲新概念，而是在补整个知识基础设施的第三层：

- `Spec`：项目契约
- `RAG`：动态知识
- `MCP`：统一访问外部工具、资源和 prompt 的接口

这个组合的含义是：

- 知识不只是“能被检索”
- 还要“能被调用”
- 而且这种调用最好有统一协议，而不是每接一个系统都手写一套胶水逻辑

所以它实际上在把“AI 程序员的上下文”从静态文档，扩展成“文档 + 检索 + 工具协议”的联合运行环境。

## 我的判断与保留

这篇的最大价值，是把几个常被分开讲的概念拼回到一起：

- `SDD / Spec`
- `RAG`
- `MCP`

从而构成一种比较完整的“AI coding 认知基础设施”视角。

但它也有明显限制：

- 后半部分的 RAG 与 MCP 介绍偏通识综述，工程深度不如前半段 framing
- 它更像知识体系设计稿，而不是已经被严格验证的大规模生产方案
- “Spec + RAG” 这个组合思路是对的，但文章本身没有给出足够多的真实项目验证细节

所以更准确地说，这篇值得保留的是架构拆分思路，而不是其中每一层都已经被证明是最佳实践。

## 阅读评级

🟡 值得追问 — 如果你关心的是如何给 AI 程序员补齐“真正懂项目”的知识底座，这篇很有参考价值。它最值得保留的，不是某个 RAG 细节，而是把 `Spec`、`RAG`、`MCP` 三层放进同一个工程框架里看

## 与其他页面的关联

- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 那篇偏 SDD 方法总论，这篇补的是为什么还需要 RAG 作为软上下文层
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — skill 解决能力与 SOP 的按需加载，这篇更偏知识与资源的双层组织
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — memory 偏会话内外状态，这篇偏项目级规范和外部知识的联合供给
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 一篇强调文档是源码，这篇进一步解释文档之外还需要什么知识基础设施来支撑 AI coding
