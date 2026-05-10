---
tags: [摘要, ai, agent, skills, context, architecture]
sources: [raw/ai/2026-05-09-agentscope-skills-progressive-disclosure-webclip.md]
updated: 2026-05-09
---

# AgentScope Skills 与渐进式披露

**来源：** [天雨、远云、刘军, 2026-02-04](https://mp.weixin.qq.com/s/_iUHLFap_3djHXMJClnfRg)

## 核心结论

这篇文章最值得保留的，不是“AgentScope 也支持 skill 了”这个产品消息本身，而是它把一个长期存在但常被混淆的问题讲清楚了：`agent` 的能力不该一股脑塞进上下文，也不该只能在“全量加载”和“RAG 碎片召回”之间二选一。

它提出的 `渐进式披露` 本质上是一种能力与上下文的分层加载机制：

- 启动时只知道“有什么能力”
- 需要时再加载完整 SOP
- 具体执行时再按需取资源或脚本

这个设计很适合作为“多能力 agent 如何避免上下文过载”的中间路线。

## 它在解决什么问题

文章设定的问题很典型：我们希望一个 agent 同时掌握很多能力，但任意一次任务通常只会用到其中一小部分。

作者把常见方案分成三类：

- `全量加载`：简单，但 token 爆炸
- `多 agent`：做了隔离，但每个 agent 内部仍可能是全量知识
- `RAG`：按需取知识，但流程型知识容易碎片化和失真

这里最重要的判断是：问题本质不是“模型不够大”，而是缺少一种能够在保持知识完整性的同时，又支持按需加载的上下文管理机制。

## Skill 在这篇里的定义很清楚

文章把 `Skill` 定义成一个独立的能力单元，通常包含三层：

- `结构化指令`：Markdown 写的 SOP
- `资源文件`：参考文档、模板、示例
- `可执行脚本`：确定性操作代码

这点和 [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] 里对 skill 的理解是高度一致的，只是那篇更偏工作习惯，这篇更偏框架级实现。

## 渐进式披露是这篇真正的贡献

文章把上下文加载拆成三层，这个拆法很值得记：

### 1. 元数据加载

启动时只注入每个 skill 的：

- 名称
- 描述
- skill id

此时 agent 只知道“有哪些能力”，并不知道每个能力如何具体执行。这样可以用很低的 token 成本注册大量 skill。

### 2. 指令加载

当模型判断某个 skill 相关时，再加载 `SKILL.md` 的完整 SOP。

这一步解决的是：

- 不用预先把所有流程知识塞进系统提示
- 又不会像 RAG 那样只拿到碎片片段

所以它更适合流程型知识、规则性知识和带顺序约束的任务。

### 3. 资源加载

执行到具体环节时，再按需取：

- reference 文档
- templates
- scripts
- 绑定 tool

这一步把“能力发现”和“能力执行资源”继续拆开了，进一步降低常驻上下文压力。

## 这篇对 tool 的处理也很关键

文章里一个很有价值的实现点是：`Tool` 也可以作为 skill 的一种资源按需暴露。

也就是说：

- skill 未激活时，绑定 tool 不在 agent 工具列表里
- skill 激活后，相关 tool 才自动变得可见

这其实是在做“工具层的渐进式披露”。它和上下文压缩一样重要，因为很多 agent 系统不是知识过多，而是工具表过大、过早暴露，导致模型选择混乱。

## 和现有页面的关系

这篇最适合作为“agent 能力组织与上下文加载”主题页，因为它把几篇现有页面里分散出现的点串了起来：

- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] 讨论状态与记忆如何分层，这篇讨论能力与指令如何分层暴露
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 强调仓库内真实约束和文档入口地图，这篇提供了更细粒度的能力打包与加载机制
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] 把 skill 讲成团队资产，这篇补的是 skill 在框架层如何被发现、加载和调度

## 我的判断与保留

我认为这篇最大的贡献，不是提出一个全新概念，而是把“progressive disclosure”正式化成了 agent 框架级能力管理模式。

但它也有边界：

- 这是能力组织机制，不直接保证任务质量
- 如果 skill 本身的 SOP 很差，按需加载只会更高效地加载坏流程
- 它比纯 RAG 更适合流程知识，但对开放式知识探索未必是替代关系

所以更准确地说，这篇解决的是“能力如何以低上下文成本被正确发现和完整执行”，不是“如何让 agent 自动变聪明”。

## 阅读评级

🟡 值得追问 — 如果你关心的是多能力 agent 怎样同时兼顾可扩展性、上下文成本和执行稳定性，这篇很值得保留。它最有启发性的地方，是把 skill 从“提示词文件”提升成了分层加载的能力管理单元

## 与其他页面的关联

- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 前者偏状态与记忆分层，这篇偏能力与 SOP 分层
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 那篇强调入口地图和仓库记录系统，这篇补的是能力包的加载机制
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 一篇偏个人与团队如何把 skill 当资产，这篇偏框架如何让 skill 真的按需生效
- [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] — `Ralph` 关注循环推进，这篇关注每轮推进时上下文和工具如何不被过度暴露
