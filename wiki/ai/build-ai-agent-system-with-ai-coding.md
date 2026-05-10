---
tags: [摘要, ai, coding, agent, workflow, architecture]
sources: [raw/ai/2026-05-09-build-ai-agent-system-with-ai-coding-webclip.md]
updated: 2026-05-09
---

# 用 AI Coding 快速打造 Agent 系统

**来源：** [柏慕, 2026-02-09](https://mp.weixin.qq.com/s/SzMyIu6pAHVvU7nUnQSwjg)

## 核心结论

这篇文章最值得保留的地方，是它把“AI Coding 怎么真正用于搭建 Agent 系统”讲成了一个具体迁移案例，而不是泛泛地说“AI 可以帮你写代码”。

核心不是单个模型能力，而是如何把：

- `LangGraph`
- `Agent Skills`
- `A2A`
- `MCP`
- `AI Coding`

组合成一套能从低代码流程平台迁移出来的工程化方案。

## 它在解决什么问题

文章面对的是一个电商导购场景：运营用自然语言描述购物场景，系统自动生成标题、标签、文案、匹配商品，并支持多轮交互、商品补全、商品过滤和外部知识接入。

原有方案基于低代码流程编排，早期验证业务可行性足够快，但随着复杂度增加，开始暴露出几个问题：

- 复杂条件分支难以维护
- 错误处理和异常恢复不灵活
- 和企业内服务深度集成时扩展能力不够
- 多轮状态和上下文管理开始变得脆弱

所以迁移目标并不只是“换个框架”，而是从单体流程编排走向模块化、多协议、可扩展的 agent 工程架构。

## LangGraph 在这里不是“潮流选型”，而是状态与流程控制底盘

文章对 `LangGraph` 的价值判断比较务实：

- 支持循环与记忆
- 能做细粒度跳转控制
- 用共享 state 管理节点间数据
- 可以在复杂条件下做稳定路由

这意味着它更适合承接多轮对话、工具调用、多 Agent 协作和跨节点状态持久化，而不是只做一次性链式流程。

和 [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] 放在一起看，这篇相当于给那张企业 AI 总图补了一张更接地气的落地剖面图。

## Skills + Planner 是这篇里最值得记的组合

文章里最重要的架构点，不是单独的 skill，也不是单独的 planner，而是两者组合起来的工作方式：

- `Skills` 负责把工具按业务领域封装成可复用能力单元
- `Planner` 先输出完整计划，再按计划决定需要激活哪些 skill

这样做的收益有三层：

- 工具不再平铺在一个大列表里
- 按需加载 skill，减少上下文膨胀
- 先规划后执行，减少步骤遗漏和顺序混乱

这一点和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] 非常互补：那篇更偏能力按层暴露的机制，这篇展示了 skill 在一个具体 LangGraph agent 项目里如何和 planner 协同。

## AI Coding 在这里的真正角色不是“自动补全”，而是迁移加速器

文章最有实用价值的一段，是把 AI Coding 具体用在了 DSL 到代码的迁移链路里：

- 先让 AI 分析线上 DSL 文件
- 根据 DSL 和平台文档设计新工程架构
- 再生成状态定义、节点函数、路由函数、图结构和测试代码

也就是说，AI Coding 在这里的价值不是帮你写某个函数，而是帮你把“旧系统的流程语义”翻译成“新系统的工程实现”。

这比常见的“让 AI 写几个接口”要高一个层级，也更接近真实重构场景。

## 这篇对提示词、文档和 rules 的用法也很工程化

文章里一个成熟点是：它不是裸用 AI Coding，而是先准备一组结构化文档和约束：

- 平台文档
- 内部 API 文档
- LangGraph 文档
- DSL 转 LangGraph 的提示模板
- `.cursorrules`

这说明在这种项目里，AI Coding 的效率高度依赖于“喂给它什么上下文”。这一点和 [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] 的判断完全一致：模型只是执行器，真正决定上限的是知识底座和约束层。

## 我的判断与保留

我认为这篇最值得保留的，不是“几天内完成重构”这种速度叙事，而是它给出了一个很具体的组合式落地范式：

- 用 `LangGraph` 解决状态与流程
- 用 `Skills` 解决能力分组与复用
- 用 `Planner` 解决多步骤任务完整性
- 用 `MCP / A2A` 解决协议与系统互联
- 用 `AI Coding` 解决迁移和重构成本

它的限制也很明显：

- 文章更多是项目复盘，不是抽象度很高的方法论文
- 一些效果判断偏经验性，缺少严格对照数据
- 低代码迁移到图工作流的收益在别的团队未必同等成立

所以更准确地说，这篇是一个高信息密度的工程样本，而不是通用模板。

## 阅读评级

🟡 值得追问 — 如果你关心的是 AI Coding 如何真正参与 agent 系统重构与搭建，这篇很有参考价值。它的亮点不在某个单点技巧，而在于展示了多个 agent 时代组件如何被组装成可落地工程方案

## 与其他页面的关联

- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 那篇给总图，这篇给一条更具体的工程化迁移样本
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 前者讲 skill 的加载机制，这篇讲 skill 在实际业务 agent 里的组织与应用
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — 那篇偏知识底座，这篇偏如何把知识、规则和 DSL 迁移真正喂给 AI Coding 工具
- [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] — `Ralph` 关注持续执行控制，这篇更偏复杂工作流搭建和多组件编排
