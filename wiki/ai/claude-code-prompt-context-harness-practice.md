---
tags: [摘要, ai, agent, claude-code, prompt, context, harness]
sources: [raw/ai/2026-05-10-claude-code-prompt-context-harness-practice-webclip.md]
updated: 2026-05-10
---

# 深度解析 Claude Code 在 Prompt / Context / Harness 的设计与实践

**来源：** [飞樰, 2026-04-20](https://mp.weixin.qq.com/s/YgGW92VBP8s846yzIxjVWQ)

## 核心结论

这篇文章和现有的 Claude Code runtime 拆解页不同，它不是优先讲启动链路和 Query Loop，而是把 Claude Code 放进 `Prompt / Context / Harness` 三层框架里重新解释。价值在于：它能帮助把 Claude Code 从“顶级 CLI 产品”翻译成“可迁移的方法论样本”。

## Prompt 层的重点不是写词，而是动态装配体系

文章强调，Claude Code 的 system prompt 不是一段固定文本，而是：

- 静态底座
- 动态模块
- 系统上下文注入
- 用户上下文注入
- 按优先级选择最终 prompt

这个判断和 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] 是互补关系：那篇更偏 runtime 分层，这篇更偏“运行前如何把 prompt 制度装起来”。

## 它对 `CLAUDE.md` 这一层的解释很有用

文中把 `CLAUDE.md` 看成 Claude Code 的项目说明书和行为规范，并区分了几类位置：

- 全局偏好
- 项目共享规范
- 本地私有指令
- 按文件类型拆分的规则

这部分很值得保留，因为它把很多人对 `CLAUDE.md` 的理解，从“单个规则文件”推进成了 `分层项目上下文系统`。这与 [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] 中的规则文件、skills、hooks 视角可以互补着看。

## Context 层最关键的是三层渐进式压缩思路

文章对 Claude Code 的 context management 重点放在：

- MicroCompact
- Session Memory Compact
- 更激进的更高层压缩

虽然这类分层未必是官方术语，但抽象方向是成立的：成熟 coding agent 不会只靠大窗口硬扛，而是要有渐进式压缩和记忆替代机制。

这也和 [[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]] 很适合对照：

- OpenClaw 更偏长期个人环境和文件记忆
- Claude Code 更偏代码任务中的会话压缩、项目规则注入和上下文节流

## 子 Agent Prompt 这一段，说明多 Agent 难点在通信协议

文中提到 Claude Code 给主 agent 的子 agent 委派提示，其意义不是“能调起 subagent”，而是：

- 什么时候该委派
- 怎么描述任务
- 如何防止乱派
- 如何控制通信带宽和摘要质量

这一点和 [[wiki/ai/chat-window-to-multi-agent-console-collaboration-shift|从聊天窗口到多 Agent 控制台]] 有共同主题：多 agent 的难点并不只是开并发，而是委派协议、摘要质量和边界治理。

## Harness 层的价值，是解释为什么 Claude Code 在同模型下也更“能干活”

文章有个隐含判断很重要：同样的 Claude 模型，在 Claude Code 里表现更强，不只是模型本身，而是外层工程设计给它“增色”。

换句话说，Claude Code 的优势很大一部分来自：

- prompt 装配
- 项目上下文文件系统
- 上下文压缩
- 运行时约束
- 任务委派机制

这个结论很适合拿来反驳“只要 API 接同一个模型就能复刻产品体验”的幼稚想法。

## 我的判断与保留

我认为这篇最值得留下的是三个判断：

- Claude Code 的很多优势来自 Prompt / Context / Harness 的协同，而不是模型独占
- `CLAUDE.md` 应该被理解成分层项目上下文体系，而不是孤立规则文件
- 多 agent 设计里，委派 prompt 和摘要协议本身就是关键工程资产

它的局限是：

- 和现有 Claude Code runtime 页有部分主题重叠
- 对一些内部机制的分层带有作者抽象色彩，不宜机械等同源码实现

## 阅读评级

🔴 建议深读 — 如果你已经看过 Claude Code 的 runtime 分析，但还想从 Prompt / Context / Harness 三层把它重新整理成可借鉴的方法论，这篇值得补

## 与其他页面的关联

- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] — 那篇偏 runtime，这篇偏 Prompt / Context / Harness 三层框架
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 一篇偏组件地图，一篇偏上下文与提示词装配机制
- [[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]] — 两篇适合对照同一三层框架在 coding agent 和 personal agent 上的不同落点
- [[wiki/ai/chat-window-to-multi-agent-console-collaboration-shift|从聊天窗口到多 Agent 控制台]] — 都触到多 agent 协作，只是这篇更偏 prompt / context / harness 设计
