---
tags: [摘要, ai, inference, llm, cache, infrastructure]
sources: [raw/ai/2026-05-09-ai-inference-kv-cache-what-we-mean-webclip.md]
updated: 2026-05-09
---

# AI 推理中的 KV Cache

**来源：** [朱云锋, 2026-02-11](https://mp.weixin.qq.com/s/N44kMpWtgmfqLB4S5MUjaA)

## 核心结论

这篇文章真正有价值的地方，不是再讲一遍 `Transformer` 入门，而是把 `KV Cache` 从“推理优化技巧”重新定义成一种 `compute cache` 基础设施。

也就是说，`KV Cache` 不是传统意义上缓存接口返回值的那类系统，而是在大模型推理过程中，缓存已经算好的 attention 中间状态，避免每次都重复做高成本前缀计算。理解这一点，才能看懂为什么后面会出现 `PagedAttention`、`RadixAttention`、`P/D Disaggregation` 和独立的 `KV Cache Layer`。

## 为什么 KV Cache 会成为核心瓶颈

文章先从 `Q / K / V` 和自注意力机制讲起，这部分虽然篇幅长，但服务的是一个关键判断：

- `Prefill` 阶段要把整段 prompt 的前缀都过一遍模型
- `Decode` 阶段每生成一个 token，都要反复依赖前面所有 token 的 `K / V`
- 如果前缀不缓存，长上下文和多轮会话的推理成本会迅速失控

所以 `KV Cache` 本质上是在用显存和系统复杂度，交换推理吞吐、时延和成本。

这也解释了为什么推理系统的很多优化，其实都围绕同一个问题展开：`怎样更高效地保存、复用、迁移和调度 KV Cache`。

## 这篇最有用的是系统视角

文章把几类常见优化串成了一条比较完整的演进链：

- `vLLM / PagedAttention`：把 KV Cache 以分页方式管理，减少连续大块显存分配造成的碎片和浪费
- `SGLang / RadixAttention`：围绕共享前缀复用 KV Cache，并让调度策略主动偏向高命中请求
- `P/D Disaggregation`：把 `Prefill` 和 `Decode` 拆到不同资源形态上，分别优化首 token 延迟和后续生成吞吐
- `LMCache`：把 KV Cache 从单一推理框架内部能力，抽成相对独立的缓存层和控制层

这条线索很重要，因为它说明业界已经不再把 KV Cache 视为某个框架里的局部技巧，而是在把它产品化、分层化、网络化。

## LMCache 这部分最值得保留

文章后半段对 `LMCache` 的总结是全篇最值得记住的部分。它把 KV Cache 的两个高价值场景说得很清楚：

- `Context Caching`：缓存公共 system prompt、RAG 上下文或复用前缀
- `P/D Disaggregation Caching`：在 `Prefill` 和 `Decode` 引擎之间高效传递前缀缓存

作者进一步指出，要把这件事做成基础设施，光有缓存数据本身不够，还需要三层能力：

- `Connector`：向上对接不同推理框架
- `Store / Load`：向下对接 CPU 内存、本地盘、网络存储等后端，并做好 batching、pipeline 和 zero-copy
- `Controller / Metadata`：维护全局 token 映射、查询、pinning、清理、迁移和压缩等控制能力

这意味着 KV Cache 的难点已经从“怎么存”转向“怎么管”。一旦进入分布式部署，真正复杂的是元数据一致性、路由和跨节点传输，而不是某个 page 的布局细节。

## 和现有知识库的关系

这篇适合和 [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] 一起看。那篇把 `Memory`、`Gateway`、`Sandbox`、`Observability` 放进企业 AI 基础设施总图里，这篇可以补上其中常被轻描淡写的一层：`Inference Runtime` 为什么会演变出独立缓存基础设施。

它和 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] 也容易混淆，但两者不是一回事：

- `Agent Memory` 解决的是跨会话状态、偏好和经验沉淀
- `KV Cache` 解决的是模型推理过程中的中间计算复用

前者更接近语义层和产品层状态，后者更接近运行时和系统层状态。

## 我的判断与保留

这篇的问题是前半段 Transformer 铺垫偏长，信息密度不稳定；但后半段一旦进入 `vLLM / SGLang / LMCache`，价值会明显上升。

我认为最值得保留的结论有三个：

- `KV Cache` 应该被理解为推理期的计算缓存，而不是普通业务缓存
- `Prefill / Decode` 分离会持续推动 KV Cache 的跨节点传输和独立化
- 真正进入生产后，KV Cache 一定会衍生出控制平面、元数据管理和专门后端，而不是永远内嵌在单个推理进程里

## 阅读评级

🔴 建议深读 — 如果你想理解大模型推理基础设施为什么会从单机显存优化，走到独立缓存层和控制平面，这篇是相当好的桥梁材料。它不是最短的入门文，但能把“模型内部机制”和“系统工程演化”连起来

## 与其他页面的关联

- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 那篇给企业 AI 应用的全栈地图，这篇补推理运行时内部为什么会长出独立缓存层
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 前者是推理期计算状态复用，后者是跨会话语义状态管理，边界要分清
- [[wiki/ai/build-ai-agent-system-with-ai-coding|用 AI Coding 快速打造 Agent 系统]] — 应用层越多 Agent 和工作流，对底层推理吞吐、时延和上下文复用的要求就越高
