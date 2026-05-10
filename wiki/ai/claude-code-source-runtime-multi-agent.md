---
tags: [摘要, ai, agent, claude-code, runtime, architecture]
sources: [raw/ai/2026-05-09-claude-code-source-runtime-multi-agent-webclip.md]
updated: 2026-05-09
---

# Claude Code 源码拆解：从启动到多 Agent 扩展层

**来源：** [无岳, 2026-04-15](https://mp.weixin.qq.com/s/VHVZV0rrCxYkbrxjuQzIAQ)

## 核心结论

这篇文章最有价值的地方，不是“带你读源码”，而是把 Claude Code 拆成了一套 `可长期承接复杂度的 agent runtime`：

- 启动链路先分流模式，再装配会话制度
- REPL 不是聊天壳，而是运行时控制面
- Query Loop 不是一次模型调用，而是显式状态机
- 工具、权限、任务、多 Agent、扩展层都被纳入统一执行链

它回答的是：为什么很多 agent 一复杂就散架，而 Claude Code 这类系统还能继续长。

## 启动层最值得学的是“先定边界，再进会话”

文章把启动链路拆成：

- 入口分流
- 进程级初始化
- 会话级准备

这个顺序的价值很高。它意味着 CLI、headless、remote、后台 session 这些宿主，不是各写各的语义，而是先共享一套 runtime / session 边界，再决定由谁承载。

这和 [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] 能连起来看：那篇更偏功能构件，这篇更强调这些构件为什么要在启动时就被制度化。

## REPL 被当成控制面，而不是消息展示器

文章对 REPL 的判断很对：一旦系统有工具调用、权限确认、后台任务、MCP、插件和远端状态，UI 就不能只是“渲染模型回复”。

Claude Code 的 REPL 更像运行时操作台，负责：

- 汇总本轮能力面
- 暴露权限和任务状态
- 归并结构化事件流
- 在进入 query 前，把这轮执行制度打包清楚

这和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 提到的“仓库与约束本身就是操作系统”是一条线，只不过这里落到了 CLI 控制面实现。

## Query Loop 被抬升成显式状态机，这才像 runtime

文章最强的一段，是把 query loop 讲成：

- 消息与工具上下文的容器
- compaction / recovery / budget 的状态机
- 工具回灌再推理的闭环

也就是说，在 Claude Code 里，模型不是整套系统，而只是运行链路里的一个节点。真正决定稳定性的，是谁在管理：

- token 预算
- 压缩和恢复
- 工具调用后的回灌
- 失败重试和中断
- 多轮 turn 的推进规则

这一点和 [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]]、[[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] 可以互相补充：后两者从方法论讲 harness，这篇从运行时骨架讲 harness 如何落地。

## 工具运行时、权限链和任务系统才是“能干活”的底座

文章把 Claude Code 的 tool runtime 视为 syscall 层，把权限系统看成整条执行链上的制度约束，这个抽象是成立的。

真正重要的不是“有多少工具”，而是：

- 工具如何注册和暴露
- 权限如何前置到执行前
- 后台任务如何不污染前台会话
- 多 Agent / subagent 如何挂在同一 runtime 语义下

这类设计说明，复杂 agent 的瓶颈很少是“再多加几个 prompt”，而是系统是否有清晰的执行边界与并发治理。

## 扩展层的意义，是把复杂度留给制度，不留给主循环

MCP、Skills、Plugins 在文中被放进统一扩展层，这是很关键的判断。它意味着扩展不是在主循环里到处加特判，而是作为标准接口接进来。

这和 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] 可以直接对照：

- Function Calling 是调用基底
- MCP 是外部系统集成协议
- Skills 是文本化 SOP
- Plugins / 扩展层则是能力挂载方式

## 我的判断与保留

我认为这篇最值得保留的三个判断是：

- agent 系统一旦进入多宿主、多工具、多任务阶段，启动层和控制面就是架构寿命线
- query loop 必须被当成状态机，而不是聊天接口封装
- 扩展能力要制度化接入，不能继续靠主循环特判兜底

它的局限是：

- 仍然是二手源码解读，不是官方设计文档
- 有些模块边界是作者抽象出来的，不必机械等同于源码命名
- 更适合拿来做 runtime 设计参考，不适合当逐文件源码索引

## 阅读评级

🔴 建议深读 — 如果你关心的不是“Claude Code 有哪些功能”，而是“复杂 agent runtime 为什么不该写成一个大主循环”，这篇值得读

## 与其他页面的关联

- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 那篇给构件地图，这篇补 runtime 分层与执行链
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 这篇能帮助理解 Claude Code 扩展层为什么必须分层
- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] — 一篇讲仓库级 harness，一篇讲 CLI runtime 级 harness
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 都在回答 agent-first 工程的控制面如何组织
