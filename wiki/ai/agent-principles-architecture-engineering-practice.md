---
tags: [摘要, ai, agent, architecture, harness, workflow]
sources: [raw/ai/2026-05-10-agent-principles-architecture-engineering-practice-webclip.md]
updated: 2026-05-10
---

# 你不知道的 Agent：原理、架构与工程实践

**来源：** [侑夕, 2026-04-28](https://mp.weixin.qq.com/s/cIQYl9Wr1Eov4ma-_bYh-w)

## 核心结论

这篇文章像一份 `agent engineering 总复盘`。它不是只讲某个框架，而是把 agent loop、workflow vs agent、控制模式、上下文分层、评测、可观测性和 harness 统成了一张总图。

## 它对 Agent Loop 的判断很稳

文中强调主循环本身很小，复杂度通常不该塞进循环体，而应放在工具、状态外化、上下文管理和外部控制面上。这和 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] 的 runtime 判断高度一致。

## Workflow / Agent 边界和五种控制模式适合当总目录

文章把 Prompt Chaining、Routing、Parallelization、Orchestrator-Workers、Evaluator-Optimizer 串起来，很适合作为现有多篇架构选型文的上层索引页。它和 [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]]、[[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] 可以互相补位。

## 上下文分层与 Prompt Caching 的那部分也值得留

它把常驻层、按需加载、运行时注入、记忆层、系统层拆得比较清楚，也把 Prompt Caching 为什么要求稳定前缀讲明白了。对理解为什么短 AGENTS、按需 Skills、外部系统约束重要，很有帮助。

## 我的判断与保留

我认为这篇最值得留下的是三个点：

- 适合作为 agent engineering 的总览页，而不是单点技巧页
- 把复杂度应落在哪里讲得比较清楚：不在 loop 内，在外部控制面
- 把上下文分层、控制模式和 harness 串成了一条连贯主线

## 阅读评级

🔴 建议深读 — 如果你想要一页能把 agent loop、架构模式、上下文工程和 harness 一次性串起来的总览，这篇很合适

## 与其他页面的关联

- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] — 都强调复杂度不应塞进主循环
- [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]] — 那篇偏企业模式地图，这篇偏原理总览
- [[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]] — 都在讨论上下文分层和 harness
