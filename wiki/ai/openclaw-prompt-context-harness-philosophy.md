---
tags: [摘要, ai, agent, openclaw, prompt, context, harness]
sources: [raw/ai/2026-05-09-openclaw-prompt-context-harness-philosophy-webclip.md]
updated: 2026-05-09
---

# 深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践

**来源：** [飞樰, 2026-04-13](https://mp.weixin.qq.com/s/JycTfNd7EnmWCnJK-QCf0Q)

## 核心结论

这篇文章的价值，在于它没有把 OpenClaw 当成“又一个爆火 agent 项目”，而是把它抽象成三层：

- Prompt Engineering：如何结构化表达约束
- Context Engineering：如何决定每轮让模型看什么
- Harness Engineering：如何设计长期运行环境

这个三段式非常适合拿来做 agent 设计复盘框架。

## Prompt 不再是一段话，而是一套动态组装机制

文章把 OpenClaw 的 system prompt 看成运行时拼装出来的模块集合，而不是手写的一大段提示词。这里最值得记的是：

- full / minimal / none 三种 prompt mode
- 按顺序拼装身份、规则、工具、workspace、memory recall 等模块
- 用 Markdown 文件体系把很多约束从代码里解耦出来

这和 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：从启动到多 Agent 扩展层]] 的思路很接近：复杂 agent 的关键不是 prompt 写得多花，而是 `运行前如何把制度装配好`。

## 文件驱动的 prompt 体系，本质是在把“身份与规则”外部化

文中反复强调：

- `AGENTS.md`
- `SOUL.md`
- `USER.md`
- `TOOLS.md`
- `BOOTSTRAP.md`
- `HEARTBEAT.md`

这些文件共同构成 agent 的人格、边界、工作方式和周期性动作。

这类设计的最大价值是：

- 可编辑
- 可审计
- 可渐进式演化
- 可按会话场景选择性加载

它和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]]、[[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] 是同一条线：把复杂能力拆成可被按需加载的文本化制度。

## Context 的核心不是“大窗口”，而是信息分层与生命周期管理

文章对 Context Engineering 的强调很到位。重点不是窗口越来越大，而是：

- 哪些信息常驻 system prompt
- 哪些作为技能按需注入
- 哪些走 memory recall
- 哪些进入 shared context
- 哪些应该被 compaction、pruning、reset

也就是说，context 不是“把资料都塞进去”，而是做信息架构设计。这个判断和 [[wiki/ai/openclaw-practice-six-agents-mac|OpenClaw 实战：六个 AI Agent]] 里那种 session 膨胀、规则失效、上下文腐烂的案例是能对上的。

## Harness 才是让 agent 长期可控的那一层

文章把 Harness Engineering 视为 Prompt / Context 之后更上层的环境设计，这个抽象很有用。它关心的不是模型这一轮说什么，而是：

- runtime 提供哪些工具和权限
- hook 在什么时候触发
- compaction / pruning / reset 如何自动发生
- 安全边界和 guardrail 如何制度化
- 长期运行中的熵增如何被压制

这一点和 [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]]、[[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] 是强关联页。前者给宏观判断，后者给 repo 级落地，而这篇提供 OpenClaw 语境下的集成样本。

## OpenClaw 值得学的，不是“养虾”，而是三层联动

文章真正有用的判断是：Prompt、Context、Harness 不能分开看。

- 只有 Prompt，没有 Context 分层，规则会被噪声淹没
- 只有 Context，没有 Harness，系统会在长期运行中失控
- 只有 Harness，没有清晰 Prompt / Context 结构，自动化会放大混乱

这使得 OpenClaw 的价值不只是功能堆叠，而是把这些层连接起来，形成一个可长期运行的 personal agent environment。

## 我的判断与保留

我认为这篇最值得留下的是三个判断：

- Prompt Engineering 在成熟 agent 里已经演化成动态系统提示词装配
- Context Engineering 的本质是信息架构和生命周期治理
- Harness Engineering 才是把 agent 从 demo 拉到长期运行系统的关键层

它的局限是：

- 方法论色彩比较强，很多抽象更适合指导设计，而不是直接指导编码
- 会和 OpenClaw 架构上下篇有一定主题重叠，但这篇胜在视角更统一

## 阅读评级

🔴 建议深读 — 如果你想要的不是 OpenClaw 模块清单，而是一个适合迁移到自己系统里的 agent 设计框架，这篇很值

## 与其他页面的关联

- [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]] — 上篇给系统拓扑，这篇给 Prompt / Context / Harness 的方法论框架
- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — 下篇讲 memory、session、skills、routing 等机制，这篇解释这些机制为什么要这样组织
- [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] — 一篇宏观论断，一篇 OpenClaw 具体化
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 都在强调按需加载与上下文节流
