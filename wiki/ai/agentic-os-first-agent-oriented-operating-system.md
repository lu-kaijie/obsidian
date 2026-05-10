---
tags: [摘要, ai, os, agent, infrastructure, platform]
sources: [raw/ai/2026-05-09-agentic-os-first-agent-oriented-operating-system-webclip.md]
updated: 2026-05-09
---

# 阿里云发布 Agentic OS：首个面向 Agent 的操作系统

**来源：** [阿里云开发者, 2026-03-31](https://mp.weixin.qq.com/s/1YaymfF3E-8MbdwOy_Ov_g)

## 核心结论

这篇文章最值得保留的，不是“首个面向 Agent 的操作系统”这个营销口号，而是它试图把一类已经在 agent 产品里分散出现的能力，整体上提到 `OS / platform` 层：

- 预置 skills
- agent-friendly shell
- token 可观测
- 运行时隔离
- 系统级安全防护

它背后的判断是：当计算负载从传统软件转向 agent，操作系统不再只是给人类和进程服务，而要开始为 `智能体负载` 提供默认运行环境。

## 它想解决的是“agent 上岗成本太高”

文章对问题的定义很直接：

- 传统 OS 对人类友好，但不天然对 agent 友好
- agent 需要花大量 token 探测环境、学习命令和适配技能
- 部署、调优、监控和安全链路太长

所以 Agentic OS 的思路不是再做一个 agent 框架，而是把高频的系统管理和运维能力提前封成环境内能力，让 agent 少花 token 做环境摸索。

这和 [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] 可以接起来看：如果 Harness 是环境治理理念，这篇就是把一部分 Harness 直接产品化、平台化。

## 三个关键词：Skill 化、Shell 化、系统级安全

文章的三大卖点可以压缩成三件事：

- `Skill 化`
  把 Linux 运维、部署、调优等高频动作做成原生 skill，减少 agent 的环境探索和 token 消耗
- `Shell 化`
  用 `Copilot Shell (cosh)` 作为新的交互入口，让人和 agent 都通过同一系统壳层调度能力
- `系统级安全`
  用 AgentSecCore、沙箱、签名校验、宿主隐私保护和安全基线扫描，把 agent 风险控制前移到 OS 层

这说明它想定义的不是“一个更会聊天的系统助手”，而是 `agent-native runtime environment`。

## 这篇真正有价值的地方在平台分层视角

文章里最值得记的，不是 30% 或 60% token 节省数字本身，而是它给出的一种平台分层：

- 核心层 / runtime 层负责受控执行
- 内置 skills 负责通用能力复用
- `cosh` 负责统一交互入口
- 可观测和保障服务负责 7x24 运行

这个分层和 [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]]、[[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] 形成了一个很好的对照：

- OpenClaw 更像个人 agent OS / control plane 产品
- Agentic OS 更像云厂商视角下，为这类系统提供底座的宿主环境

## Token 可观测是一个容易被忽视的重点

文章里一个相对少见但值得记的点，是它把 token 统计做到系统层，并区分：

- system prompt
- skills 注册表
- history

等不同 token 来源。

这很重要，因为一旦 agent 进入生产环境，很多优化问题最终都会变成：

- token 花在哪
- 哪些环境信息在浪费上下文
- 哪些内置能力在降低探索成本

这与 [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] 的思路一致：上下文不是越多越好，而是要可拆解、可归因、可优化。

## 我的判断与保留

我认为这篇最该保留的是它的 `基础设施升级信号`：

- agent 正在被当成新的计算负载类型
- runtime、安全、skills、可观测性开始被统一打包到底层平台

它的局限也很明显：

- 这是发布稿，技术细节和外部验证还不够多
- 大量收益数据来自平台自述
- “Agent OS” 这个命名本身带有较强产品 framing

所以它更适合当 `平台趋势页`，而不是严格技术白皮书。

## 阅读评级

🟡 值得追问 — 如果你关心的是 agent 能力为什么会开始向操作系统和云平台层下沉，这篇值得留。它最有价值的是平台视角，而不是单个指标

## 与其他页面的关联

- [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]] — OpenClaw 展示 personal agent system 长什么样，这篇展示云平台想为这类系统提供怎样的宿主环境
- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — 下篇讲 sandbox、memory、session 等运行机制，这篇把类似诉求上提到 OS 级能力
- [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] — Harness 讲环境治理理念，这篇更像把治理能力产品化成平台底座
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 一篇讲 skill 如何被组织给 agent 使用，这篇讲 skill 如何作为系统内置能力下沉到平台层
