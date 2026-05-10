---
tags: [摘要, ai, memory, agent, architecture, theory]
sources: [raw/ai/2026-05-09-memory-architecture-ledger-views-policy-webclip.md]
updated: 2026-05-09
---

# 「纯干货」几万字都讲不明白的Memory架构与思考

**来源：** [陈梓康（库达）, 2026-04-07](https://mp.weixin.qq.com/s/bl77_Mb85C4AKe8h4__V6Q)

## 核心结论

这篇文章最值得保留的，不是它罗列了很多 2026 年初的 memory 论文，而是它试图把 Memory 从“一个向量库或聊天记录”提升成一套更硬的系统抽象：

- `Raw Ledger`
- `Derived Views`
- `Policy`

也就是：

- 原始事件账本
- 面向检索与推理的派生视图
- 控制读写、更新、遗忘和回放的策略层

这是目前库里关于 memory 最完整、也最偏“系统本体论”的一篇。

## 它最重要的判断：Memory 不是存储，而是“历史影响当前决策的通道”

文章首先把问题立住了：

- 存了很多历史，不等于有记忆能力
- 只有当历史能在当前状态下影响决策分布，Memory 才真正成立

所以作者认为：

- Memory 不是“存档”
- 而是“把历史转成当前可用信息”的机制

这个 framing 很关键，因为它把 memory 从静态数据问题，改写成了 `决策接口` 问题。

和 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] 相比，那篇更偏分层综述；这篇更进一步，直接问“记忆能力的最小闭包到底是什么”。

## Ledger / Views / Policy 这组三件套很有保留价值

作者提出的最强抽象，是把 Memory 的最小闭包定义成三件套：

- `Raw Ledger`
  追加式权威记录，保证审计、回放、回滚和 provenance
- `Derived Views`
  向量索引、keyword/hybrid、KG/TKG、timeline、skill index 等派生状态
- `Policy`
  决定什么时候读、读多少、什么时候写、怎么更新、怎么遗忘

这个拆法的价值在于，它明确承认：

- 单独 event 流不可用
- 单独向量库不可治理
- 单独 prompt 控制不可复现

所以 Memory 不是一个组件，而是一个闭环系统。

## 它把 Memory OS 讲成了一个显式的 System 2

文章进一步提出：

- `System 1` 负责通用推理和 agent 能力
- `System 2` 负责 memory 的写入、检索、过滤、更新和慢回路控制

这套分工的好处是：

- 不把所有记忆能力硬塞进模型权重
- 记忆层可插拔、可迁移、可归因
- 不容易因为记忆特化训练伤到通用 agent 能力

这个思路和 [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] 的文件优先记忆层能对应上，但这篇更抽象、更像 memory OS 理论稿。

## 非参数化逼近参数化，是这篇真正的核心野心

文章没有停在“做个外部记忆库”上，而是在问：

- 非参数化 memory 能不能尽量逼近参数化 memory 的效果上限？

它给出的方向是：

- 工具化 memory 操作
- 让 agent 学会主动控制 ADD / UPDATE / DELETE / FILTER / SUMMARY
- 借助 RL 或 agentic protocol 训练 policy
- 用 PreThink / Retrieve / Write / Early Stop 等机制降低不必要检索

也就是说，这篇的目标不是“把记忆外挂到 agent 上”，而是“让外挂记忆层越来越像模型内部能力，但仍保留可观测和可治理优势”。

这个判断很重要，因为它比一般 RAG/Memory 文章更接近长期路线图。

## 时序记忆部分也值得单独保留

文章后半段对 temporal memory 的讨论很强，核心是：

- 时间不是 metadata，而是结构维度
- 需要区分 `valid time` 和 `transaction time`
- current / history / all 的检索语义应该是策略层显式控制的

这意味着 memory system 不该只关心“相关不相关”，还要关心：

- 这件事什么时候为真
- 什么时候被系统知道
- 什么时候被纠正或撤销

这部分和一般 memory 文章差异很大，已经开始接近数据库、时序图谱和认知系统的混合设计。

## 我的判断与保留

我认为这篇最该保留的，不是其中每篇论文结论，而是它沉淀出的三个框架判断：

- Memory 是决策通道，不是存储桶
- 非参数化 Memory 的最小闭包是 `Ledger + Views + Policy`
- 一个可治理的 memory system，最终会长成显式的 `System 2`

它的局限也很明显：

- 文章很长，论证方式偏研究札记，不是严格论文
- 许多论文仍然很新，外部验证不足
- 某些推导是高水平工程直觉，不是已经被广泛验证的工业共识

但这不影响它作为当前知识库里最强的一页 memory 理论框架被保留。

## 阅读评级

🔴 建议深读 — 如果你关心的不是“怎么接一个向量库”，而是“Memory 在 Agent 系统里到底应该被建模成什么”，这篇非常值得读。它给出的 `Ledger / Views / Policy` 抽象足够底层，也足够可延展

## 与其他页面的关联

- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 那篇做总览分层，这篇把 memory 的最小系统闭包推到了 `Ledger / Views / Policy`
- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — OpenClaw 给出文件优先记忆实现样本，这篇给出更一般化的 memory OS 理论框架
- [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] — 一篇偏工程知识底座，这篇偏长期记忆底座；两者都在回答“外部状态如何成为 agent 能力”
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — Spec / RAG 更多偏项目知识供给，这篇讨论的是跨时序、跨会话、可回放的记忆控制系统
