---
tags: [摘要, ai, coding, harness, engineering, workflow]
sources: [raw/ai/2026-05-10-ai-coding-rate-90-harness-engineering-webclip.md]
updated: 2026-05-10
---

# Harness Engineering：耗时一周，我是如何将应用的AI Coding率提升至90%的

**来源：** [新安, 2026-05-07](https://mp.weixin.qq.com/s/rlIyIIZOXFObNIXbPI7gDg)

## 核心结论

这篇文章的价值不在“90% AI coding rate”这个数字，而在它把企业存量 Java 应用里的 Harness 落成了一套相对完整的目录和操作系统：`rules + skills + wiki + changes`。

## 它把 Failure Modes 和落地手段直接对上了

文中沿着 Anthropic 常见失败模式展开，比如一步到位、过早宣布完成、缺乏跨会话记忆、自评不可靠，然后用本地实践去对冲。这比抽象谈 Harness 更有用，因为它说明每条约束都应该对应一种具体失效模式。

## 四要素架构是这篇最值得记的部分

作者把 Harness 拆成：

- `rules`：稳定约束
- `skills`：可复用 SOP
- `wiki`：领域和项目知识
- `changes`：变更追溯链

这个拆法和 [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] 很接近，但更偏“在一个复杂老项目里怎么分层补齐”。也和 [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] 的概念层能形成上下呼应。

## Application Owner 这个编排角色值得注意

文章不是让所有 agent 平铺直叙地工作，而是引入一个应用 owner 来串联需求、知识、技能和验证。这说明大规模 harness 不只是目录设计，还需要一个顶层编排者来维持节奏和验收。

## 我的判断与保留

我认为这篇最值得留下的是三个点：

- 把失败模式、约束和目录结构直接映射起来
- 说明存量项目里提升 AI 编码率，关键是显化隐性知识而不是换模型
- 给出了一套比“写个 AGENTS.md”更厚但仍可执行的 Harness 骨架

## 阅读评级

🔴 建议深读 — 如果你关心的是“老项目里如何系统化把 AI coding rate 从局部试验推进到主流程”，这篇值得读

## 与其他页面的关联

- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] — 一篇偏仓库机制，一篇偏复杂 Java 应用落地
- [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] — 一篇给概念辨析，一篇给实战目录
- [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] — 都在说明隐性知识必须外化
