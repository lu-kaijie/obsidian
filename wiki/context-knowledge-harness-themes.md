---
tags: [分析, ai, context, knowledge, harness, coding]
sources:
  - wiki/overview.md
  - wiki/ai/effective-context-engineering-for-ai-agents.md
  - wiki/ai/claude-code-prompt-context-harness-practice.md
  - wiki/ai/openclaw-prompt-context-harness-philosophy.md
  - wiki/ai/spec-driven-development-redefining-ai-coding.md
  - wiki/ai/progressive-spec-ai-coding-practice-guide.md
  - wiki/ai/spec-rag-ai-programmer-knowledge-infra.md
  - wiki/ai/qoder-harness-engineering-guide.md
  - wiki/ai/agents-md-practice-guide.md
  - wiki/ai/ai-agent-memory-systems-architecture-practice.md
  - wiki/ai/memory-architecture-ledger-views-policy.md
updated: 2026-05-12
---

# Context、知识基础设施与 Harness 主题综述

## 任务说明

这篇文档聚焦当前知识库里最强的一条母题：`Prompt 不再是主战场，Context、Spec、RAG、Memory、Harness 才是。`

它试图回答：

- 为什么这个知识库会反复围绕 prompt/context/harness 三层打转
- 为什么 AI Coding 和 Agent 应用最后都会回到“知识基础设施”问题
- `Spec`、`RAG`、`Memory`、`Skills`、`AGENTS.md`、`Harness` 各自应该放在哪一层

## 一、这个知识库最稳定的共识：Prompt 只是很小的一层

如果必须用一句话概括这一整簇文章的共同判断，那就是：

`强系统不是靠更神的 prompt，而是靠更对的上下文和更硬的运行环境。`

[[wiki/ai/effective-context-engineering-for-ai-agents|Effective context engineering for AI agents]] 给出了最官方的总论：真正影响 agent 能力的，不只是提示词措辞，而是如何检索、组织、压缩和维护上下文。

[[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]]、[[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]] 则分别从项目型和个人环境型系统补充说：同样是 agent，系统真正拉开差距的地方不是一句 system prompt，而是：

- 让模型看到什么
- 什么时候看到
- 看到的材料是规则、摘要、检索结果还是工具回灌
- 错了之后谁来阻止、验证和恢复

这意味着从工程上看，Prompt 只是上下文包裹层，而不是系统的核心。

## 二、为什么 `Context Engineering` 会取代 `Prompt Engineering` 成为中心问题

Prompt engineering 当然没有消失，但它在知识库里的地位已经明显退到局部优化层。

原因很简单：  
当系统开始拥有工具、状态、检索、记忆、规则文档、权限和会话历史时，模型输入早就不再是一段孤立提示词，而是一个 `working set`。这时候真正决定效果的，是 working set 如何构造。

这也是为什么知识库里会同时出现这些看似不同的话题：

- `AGENTS.md`
- `Spec`
- `RAG`
- `Skills`
- `Memory`
- `Compaction`
- `Session summary`
- `Retrieval`

它们本质上都在回答同一个问题：

`模型每一轮工作时，到底该看见什么。`

从这个角度看，Context engineering 不只是“检索增强”，而是对整个输入工作集的制度化管理。

## 三、`Spec` 为什么在这套知识库里地位这么高

关于 AI Coding，知识库里有一条非常清晰的演进线：

- vibe coding
- rules
- spec
- progressive spec
- harness

[[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] 之所以重要，是因为它把 Spec 从“开发前文档”提升成了 `代码之上的控制面`。  
当代码可以被模型快速重写时，真正稀缺的已经不是实现本身，而是：

- 为什么这么做
- 边界在哪
- 做到什么算完成

Spec 把这套决策压缩成可复用的约束资产。

[[wiki/ai/progressive-spec-ai-coding-practice-guide|渐进式 Spec 实战指南]] 又提醒了另一件事：Spec 不能被教条化。对简单需求强行套完整 spec 流程，本身会制造新的偶然复杂度。所以更现实的做法是：

- Rules 常驻
- Spec 按复杂度渐进加载
- 简单问题轻流程
- 复杂问题重流程

这条线的价值在于，它让知识库对 Spec 的态度既坚定又不宗教化。

## 四、`AGENTS.md`、`docs/`、`scripts/`、`harness/` 为什么总是一起出现

[[wiki/ai/agents-md-practice-guide|AGENTS.md 实践指南]] 和 [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] 放在一起看，会得到一个非常清楚的 repo 级知识分层。

`AGENTS.md` 的职责不是承载全部知识，而是做导航地图。  
它应该告诉 Agent：

- 这个仓库是什么
- 去哪里找更细的知识
- 有哪些关键约束
- 常用命令和流程入口是什么

`docs/` 承载真实知识层。  
它存放架构、产品语义、模块说明、设计记录、执行计划等相对稳定但较长的资料。

`scripts/` 承载机械执法层。  
一旦规则只停留在文档里，就仍然依赖模型理解；一旦进入脚本和 validate 管道，它才从建议变成约束。

`harness/` 承载状态、轨迹和经验层。  
它让系统不只是“知道规则”，还记得自己如何做过、卡在哪里、什么方式有效。

这四层其实构成了一套很成熟的知识基础设施分工：

- 地图
- 正文
- 机械验证
- 运行痕迹

## 五、`Spec + RAG + MCP` 是知识供给层，而不是概念拼盘

[[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] 对知识库贡献最大的不是某个技术细节，而是它给出了一个相当清楚的三层组合：

- `Spec`：项目级硬约束
- `RAG`：动态软上下文
- `MCP`：外部能力访问层

这个拆法非常重要，因为它避免了当前很多系统里常见的混乱：把规范、历史经验、临时背景资料和外部工具全塞进一个“知识库”概念里。

更准确的做法应该是：

- 必须被遵守的东西，放进 Spec
- 需要按需补充的背景资料，放进 RAG
- 需要被调用而非被阅读的外部能力，用 MCP 或工具协议接入

这也是为什么知识库里关于 RAG 的讨论并没有停留在“向量库怎么接”，而是不断强调 query、chunking、rerank、引用质量和上下文装配。

## 六、为什么 `Memory` 不等于 RAG，也不等于聊天记录

关于 memory，这套知识库里至少有两层非常清楚的共识。

第一层来自 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]]：  
memory 至少要分成会话内工作记忆和跨会话长期记忆。它不是“多存一点消息”，而是围绕提取、检索、更新和回灌组织起来的系统能力。

第二层来自 [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]]：  
如果再往深一层抽象，memory 更像：

- `Raw Ledger`：原始事件真相层
- `Derived Views`：面向检索与推理的派生视图
- `Policy`：控制读写、遗忘、更新和回放的策略层

这个抽象很强，因为它把 memory 从“一个向量库”提升成了系统本体论的一部分。

从应用开发角度看，最实用的结论是：

- `RAG` 解决外部知识补充
- `State` 解决当前任务进度
- `Memory` 解决跨轮、跨会话复用

很多项目在长期记忆上容易过度设计，但知识库的整体倾向其实更克制：先把状态、检索和摘要做好，再决定 memory 是否值得重做。

## 七、Harness 的真正含义：把非确定性模型嵌进确定性交付链路

知识库里关于 Harness 的文章很多，但它们最终不是在重复一件事，而是在逐步把同一个定义压实。

最准确的定义来自 [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]]：  
Harness 是把非确定性模型嵌进确定性交付链路的控制面。

这个定义的重要性在于，它把 Harness 从“加点测试和规范”提升成系统设计原则。于是很多看起来分散的东西突然能串起来：

- 预验证
- validate
- verify
- 规则脚本化
- 审批流
- 任务状态
- 日志和 trace

[[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] 进一步说明，Harness 最成熟的形态不是靠人类重复提示模型，而是把质量约束前移到执行前和执行中。

所以一个成熟的 repo 级 AI coding 系统，真正关键的是：

- 动手前能不能拦住明显错误
- 执行时能不能提供明确边界
- 结束后能不能做可靠验证

## 八、把这条主线压缩成一张知识基础设施地图

如果把全库这条线压成一张最小地图，我会这样分：

`Prompt layer`

- 局部表达、语气、格式、角色、输出约束

`Context layer`

- system prompt
- AGENTS.md
- docs
- 检索结果
- 状态摘要
- 历史消息
- tool output
- session summary
- memory recall

`Knowledge layer`

- Spec
- RAG
- Skills
- memory views
- project wiki

`Execution layer`

- tools
- MCP
- scripts
- validate / verify
- sandbox

`Harness layer`

- 权限
- 审批
- 预验证
- 回归
- 观测
- 经验沉淀

从这张图就能看出：知识库为什么反复在讨论 seemingly different topics。因为它们其实都在长同一套“模型工作环境”。

## 九、这条主题线对应用开发岗最有价值的表达

如果面试官问“你怎么看 context engineering”，一个高质量答案不该只说“把相关资料放进 prompt”。更完整的说法应该是：

`我理解的 context engineering，不只是检索，而是管理模型每一轮看到的整个工作集，包括规则、状态、历史、外部知识、工具结果和压缩摘要。进一步往下，它会自然连接到 Spec、RAG、Memory、Skills 和 Harness。对 AI Coding 或 Agent 应用来说，prompt 只是外层接口，真正决定系统质量的是知识基础设施和运行环境设计。`

## 关联页面

- [[wiki/ai/effective-context-engineering-for-ai-agents|Effective context engineering for AI agents]]
- [[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]]
- [[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]]
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]]
- [[wiki/ai/progressive-spec-ai-coding-practice-guide|渐进式 Spec 实战指南]]
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]]
- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]]
- [[wiki/ai/agents-md-practice-guide|AGENTS.md 实践指南]]
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]]
- [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]]
