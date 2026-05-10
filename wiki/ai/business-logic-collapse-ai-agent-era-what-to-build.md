---
tags: [摘要, ai, agent, architecture, methodology]
sources: [raw/ai/2026-05-09-business-logic-collapse-ai-agent-era-what-to-build-webclip.md]
updated: 2026-05-09
---

# 业务逻辑的“坍塌”：当应用层只剩下胶水代码，在 AI Agent 时代，我们该构建什么

**来源：** [韩春, 2026-03-24](https://mp.weixin.qq.com/s/kVE6an0dqnO34SQnRXddWg)

## 核心结论

这篇文章最值得留下的，不是某个具体框架，而是它对一个更底层变化的判断：当 LLM 把一部分业务知识和决策模式吸进模型权重后，传统应用层里大量靠代码展开的业务逻辑，会逐渐塌缩成 `上下文组织 + 状态管理 + 工具编排 + 安全约束` 的胶水层。

所以问题不再只是“怎么调用大模型”，而是：在一个内核本身不透明、且天然不稳定的执行体之上，应用层到底还应该负责什么。

## 它把“不确定性”讲成了系统前提，不是小缺陷

文章前半段有个重要澄清：`temperature=0` 并不等于绝对确定输出。

作者把原因归到几类现实工程约束：

- 浮点精度误差和累积放大
- 异构硬件上的细微差异
- runtime 为吞吐与成本做的各种近似与优化
- 采样与路由层面的工程折衷

这里最值得记住的不是具体细节，而是这个判断：

- LLM 不确定性不是暂时 bug
- 很多时候它还是模型厂商为了成本和吞吐主动接受的 feature

这和 [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] 的主线一致：AI 工程的核心不是消灭不确定性，而是围住它、利用它、接受它。

## “AI 开发”其实是在搭一个上下文执行系统

文章中段最有价值的部分，是把一个手搓 Agent 拆成了完整工程问题，而不是 API 调用问题。

它列出来的真正工作包括：

- RAG 与检索参数
- short-term / long-term memory 管理
- prompt 角色边界与消息次序
- LLM 选型、重试、格式修复与评测
- tool 调用、失败恢复与权限控制
- 多 Agent 编排、循环检测与 step 限制
- prompt injection、防泄漏与输出治理

也就是说，所谓 AI 开发，实际是在构造一个围绕模型的不确定执行环境，而不是把聊天框换成 SDK。

这点和 [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]]、[[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] 能接起来看：前者强调流程和初始化协议，后者强调知识供给层，这篇则把这些东西统一落回“应用层到底在忙什么”。

## 它对 LangChain 的判断比常见批评更准

文章没有把 LangChain 的价值简单归结为 DAG 或 workflow 编排，而是认为它真正重要的地方，是给 AI 开发定义了语义层标准接口：

- `System / User / Assistant / Tool` 的消息边界
- 检索、记忆、模型调用、工具调用的标准组件化
- 编排循环中的状态传递与模块职责

这个判断很关键，因为很多团队会把编排形式误认为核心壁垒，但作者指出：

- `ReAct`、`workflow`、`plan-exec` 在控制流上并不神秘
- 更长期的价值在于接口标准化与组件抽象

这和 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] 是互补关系。那篇在拆工具体系的层次，这篇在解释为什么语义分层本身会成为基础设施。

## “业务逻辑坍塌”不是业务消失，而是重心迁移

作者最强的一句潜台词其实是：

- 过去复杂性大量写在业务代码里
- 现在复杂性的一部分被模型吞掉了
- 剩下的复杂性转移到了上下文、状态、路由、评测与治理层

所以应用层没有消失，只是从“显式实现业务规则”转向“组织一个足够可控的认知执行环境”。

这个变化可以和 [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] 放在一起看：

- 组织层面，问题从 ownership 转向 routing + governance
- 应用层面，问题从业务规则实现转向 context + orchestration + safety

两篇其实在描述同一件事的不同截面。

## 我的判断与保留

我认为这篇最值得长期保留的，不是它列举的某个 Agent 伪代码，而是它对 `应用层职责迁移` 的命名和解释。

它真正提出的问题是：

- 如果模型内部已经隐含了一部分“业务理解”
- 如果输出的不稳定性又无法被彻底消除
- 那我们该构建的就不再是传统意义上的厚应用层

而是：

- 可审计的上下文注入层
- 可恢复的状态管理层
- 可治理的工具与权限层
- 可验证的评测与回退层

它的局限也有：

- 后半段覆盖面很广，深浅不完全均匀
- 对 LangChain、混合架构和 LLM 性能的讨论偏综述式
- 有些推论更像资深工程师的认知压缩，而不是严格实证结论

但作为一篇把“AI 应用层到底在重构什么”说透的文章，它的保留价值很高。

## 阅读评级

🔴 建议深读 — 如果你关心的不是“怎么接一个 Agent 框架”，而是“AI 时代应用层、业务逻辑和工程职责到底迁移到哪里去了”，这篇值得反复看。它提供的是一个比工具清单更底层的解释框架

## 与其他页面的关联

- [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] — 这篇把不确定性视为工程前提，那篇提供更上层的方法论框架
- [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] — 一篇讲应用层职责坍塌，一篇讲组织和治理重心塌缩
- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 那篇偏流程编排与初始化协议，这篇解释这些控制层为何会成为主角
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — 一篇偏知识供给层，一篇偏应用执行层
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 那篇拆工具调用分层，这篇说明这些分层如何承接正在变薄的业务逻辑
