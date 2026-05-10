---
tags: [摘要, ai, agent, architecture, selection, methodology]
sources: [raw/ai/2026-05-09-agent-skills-teams-architecture-evolution-selection-webclip.md]
updated: 2026-05-09
---

# Agent / Skills / Teams 的架构选型

**来源：** [飞樰, 2026-03-17](https://mp.weixin.qq.com/s/Z8JYgxUdHSLo4ywgyt4ljg)

## 核心结论

这篇文章最值得保留的地方，是它把近两年的 agent 架构分裂重新整理成了一条比较清楚的演化链：

- `Single Agent`
- `Multi-Agent`
- `Agent Skills`
- `Agent Teams`

作者的核心判断是：这些形态不是为了“炫技”，而是为了补偿当前大模型在 `领域知识注入` 和 `长期记忆复用` 上的能力缺口。因此，架构演化史本质上就是一部“如何更便宜、更稳定地把知识和记忆外挂给模型”的工程史。

## 这篇最重要的是选型顺序，而不是概念堆砌

文章最后给出的 `P0 / P1 / P2 / P3` 顺序，非常值得直接记下来：

- `P0`：能用 `Single Agent` 解决的，绝不上复杂架构
- `P1`：遇到知识瓶颈，优先引入 `Agent Skills`
- `P2`：只有在前两者失效且追求效果上限时，再谨慎上 `Multi-Agent`
- `P3`：面对高度不确定的探索任务，再叠加 `Agent Teams`

这套排序的价值在于，它把很多“应该先堆多 agent 还是先做 skills”的争论，转成了一个更务实的奥卡姆剃刀原则：`先从最小结构开始，按瓶颈升级。`

## Single Agent 的边界说得很实在

文章对 `Single Agent` 的界定比较严格：主要指基于 `System Prompt + ReAct + 串行工具调用` 的原生单智能体模式。

作者认为它仍然适合：

- 场景复杂度低
- 领域知识体量可控
- 检索链路质量有保障

这页最值得留下的一句判断是：`长上下文不等于长记忆。`

也就是说，模型虽然能吞很多 token，但一旦靠全量知识硬塞，很容易出现 `Lost in the Middle`，导致关键信息丢失。

## RAG 和 Multi-Agent 不是天然上位替代

文章对 `RAG` 的判断很典型但仍然重要：它只是 `Single Agent` 的一次增强，不是银弹，而且高度依赖检索准确率。

对 `Multi-Agent` 的判断则更有价值。作者借 Google DeepMind 的实验结论提醒：

- 更强模型通常更好
- 但更多 agent 不一定更好
- 频繁 agent 通信会吞掉上下文预算
- 独立 agent 还可能放大错误

这和很多直觉相反。它意味着 `Multi-Agent` 的主要代价不是只多一点延迟，而是：

- 通信带宽消耗
- 错误链式放大
- 系统调优复杂度暴涨

## Skills 是这篇最推崇的中间路线

文章对 `Agent Skills` 的定位非常明确：它比 `Multi-Agent` 更轻，比 `RAG` 更精，比单纯全量 prompt 注入更稳。

作者强调的点是：

- skill 像说明书，按需加载
- 同一个主 agent 持续执行，避免信息割裂
- 通过渐进式披露控制上下文长度

这和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]]、[[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] 非常互补：前者讲技能加载机制，这篇讲为什么它在架构选型上应优先于多 agent。

## Agent Teams 的价值被放在“探索型任务”里

文章把 `Agent Teams` 和传统 `Multi-Agent` 区分开，重点不是多几个 agent，而是：

- 并行探索不同解法
- 共享任务进度和上下文
- 适合没有明确路线图的复杂问题

这个定位很关键，因为它说明 `Agent Teams` 不是通用默认架构，而更像面向未知空间搜索的一种高成本模式。

也因此，作者并没有把它放在优先路径前面，而是明确放到 `P3`。

## 这篇最有实用性的部分是 Google 论文结论映射

文章结合 Google 的 benchmark，总结出几条对工程很有用的判断：

- 降低 agent 间沟通成本
- 单 agent 成功率达到一定水平后，不要盲目加 agent
- 没有中心化纠偏时，错误会被大幅放大
- 任务类型不同，最佳架构不同

这页的价值在于，它不是只讲“经验觉得这样好”，而是尝试把经验和实验结论接起来。

## 和现有页面的关系

这篇适合作为当前知识库里 `agent 架构选型总览页`。

它和 [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] 的关系最紧：那篇更偏 workflow 与 skill 的执行编排，这篇更偏“在什么条件下应该优先用 skills，而不是上 multi-agent”。

它也能补 [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]]：那篇谈组织层 execution graph，这篇谈 agent 系统本身的结构复杂度治理。

## 我的判断与保留

我认为这篇最值得留下的是 `架构升级顺序`，而不是其中对每种模式的定义细节。

因为真正稀缺的不是知道名词，而是知道：

- 什么时候别急着上复杂架构
- 什么时候 skills 比 multi-agent 更划算
- 什么时候 teams 才值得为探索性付出并行成本

它的局限是：

- 对 Anthropic 新范式的讨论部分，仍然带有较强框架解释色彩
- 某些模式之间的边界在不同实现里会有重叠

但作为一篇选型指南，它已经足够清楚。

## 阅读评级

🔴 建议深读 — 如果你现在正在犹豫“该继续堆 workflow、上 multi-agent，还是先做好 skills”，这篇值得细读。它最大的价值不在定义概念，而在于给出了一条由简入繁的架构选型路径

## 与其他页面的关联

- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 那篇讲 workflow 和 skill 如何提升可靠性，这篇讲它们在总架构选型里为什么应优先于 multi-agent
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 一篇偏技能加载机制，这篇偏技能在架构演进中的位置
- [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] — 一篇偏组织执行图，这篇偏 agent 系统结构图
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 那篇偏 CLI 控制面能力分层，这篇偏更宏观的 agent 架构层级取舍
