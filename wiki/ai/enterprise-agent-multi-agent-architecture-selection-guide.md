---
tags: [摘要, ai, agent, enterprise, architecture, selection]
sources: [raw/ai/2026-05-09-enterprise-agent-multi-agent-architecture-selection-guide-webclip.md]
updated: 2026-05-09
---

# 企业级多智能体架构选型

**来源：** [刘军, 2026-03-20](https://mp.weixin.qq.com/s/_bz8DEgp4Lqt-xTa_lWN0A)

## 核心结论

这篇文章最值得保留的地方，是它把企业多智能体落地从“概念讨论”推进到了 `模式到场景` 的映射表。

作者的核心判断很清楚：

- 企业级 agent 不是上来就堆多智能体
- 应坚持 `Single Agent First`
- 只有跨过上下文、职责、并行和结构化流转这些复杂度阈值后，才进入多智能体模式

和前面偏总论的页面相比，这篇更像一本 `企业模式手册`：它关心的不是演化史本身，而是具体业务里该选 `Pipeline / Routing / Skills / Subagents / Supervisor / Handoffs / Custom Workflow` 的哪一种。

## Single Agent First 是这篇最重要的原则

文章一开始就强调：绝大多数日常业务需求，先用 `单智能体 + 一组精准工具`。

只有在以下四类阈值出现时，才应升级：

- `上下文管理`
- `职责分工`
- `并行加速`
- `结构化流转`

这个 framing 很实用，因为它把“为什么要多智能体”从技术偏好问题，转成了复杂度管理问题。

这和 [[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] 的 `P0/P1/P2/P3` 顺序是同一条线，但这篇更偏企业交付和框架实现角度。

## 这篇的核心价值在模式地图

文章把常见模式分得很清楚：

- `Pipeline`：固定顺序、并行、循环
- `Routing`：分类后分发给专家再合并
- `Skills`：按需披露知识，不做上下文隔离
- `Subagents`：任务委派给隔离上下文中的子智能体
- `Supervisor`：监督者把专家当工具调用
- `Handoffs`：基于状态在智能体间切换“当前负责者”
- `Custom Workflow`：用 StateGraph 自定义图式编排

这张表真正有价值的地方，不是名词解释，而是它把“谁负责决策”“上下文是否隔离”“是否保留对话历史”“是否要显式流程图”这些选型维度拆开了。

## Routing、Supervisor、Skills、Subagents 的边界讲得最有用

这篇对几个最容易混淆的模式区分得比较实战：

- `Routing`：适合一次性的分类→专家→合并，不维护长期对话记忆
- `Supervisor`：适合在持续对话里动态决定调哪个专家
- `Skills`：适合单智能体多专长，但共享上下文
- `Subagents`：适合需要隔离上下文、工具权限和职责边界的子智能体委派

这个区分很关键，因为企业系统里很多问题其实不是“功能能不能做”，而是：

- 要不要隔离上下文
- 专家是预先分发还是实时调度
- 用户在每轮对话里看到的是一个连续角色，还是多个显式交接角色

## 工作流模式 vs 对话模式，是企业系统里很重要的二分

文章把多智能体模式分成两大类：

- `工作流模式`
- `对话模式 / 自治智能体`

这条分界线对企业很重要，因为它几乎直接对应：

- 可预测性
- 成本
- 审计能力
- 调试难度

工作流模式更适合高合规、强流程、可审计场景；对话模式更适合开放式、模糊输入和高度弹性的任务。

文章给出的最佳实践是 `混合工作流`：用确定性图当脊椎，只在需要高认知灵活性的节点放入自治 agent。

这个判断和 [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] 中“企业 AI 交付不是纯模型问题，而是系统工程问题”高度一致。

## AgentScope + Spring AI Alibaba 部分，补的是实现路线

相比偏方法论文章，这篇多了一层实现指向：

- AgentScope 偏 agentic 能力
- Spring AI Alibaba Graph 偏 workflow 编排
- 两者结合，形成 `Agentic 能力 + Workflow 脊椎` 的企业方案

这意味着企业不是只能二选一：

- 不是纯工作流
- 也不是纯自治 agent

而是可以把 ReActAgent、Memory、Toolkit 这些 agent 组件，作为图中的节点被统一编排。

## 我的判断与保留

我认为这篇最值得留下的是 `模式与场景映射表`。

因为当团队真的开始做企业 agent 时，最常见的痛点不是“不知道什么是 multi-agent”，而是：

- 不知道什么时候该从单智能体升级
- 不知道 Routing 和 Supervisor 实际差在哪
- 不知道 Skills 与 Subagents 到底何时该切

这篇恰好在回答这些问题。

它的局限是：

- 偏 AgentScope / Spring AI Alibaba 生态视角
- 很多模式的优劣仍需结合实际业务约束落地验证

但作为企业选型页，它已经足够高信号。

## 阅读评级

🔴 建议深读 — 如果你已经不是在问“要不要做 agent”，而是在问“企业业务里应该上哪种多智能体模式”，这篇非常值得细读。它的价值在于把模式名字翻译成了工程决策条件

## 与其他页面的关联

- [[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] — 那篇偏总论与升级顺序，这篇偏企业场景下的具体模式地图
- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 一篇给企业 AI 系统总图，这篇补多智能体编排层的具体选型方法
- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 那篇偏 workflow 与 skill 可靠性设计，这篇偏多种 agent 模式的企业选型边界
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 那篇偏个人/CLI agent 产品抽象，这篇偏企业级 agent 编排模式
