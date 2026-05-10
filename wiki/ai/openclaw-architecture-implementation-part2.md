---
tags: [摘要, ai, agent, runtime, security, infrastructure]
sources: [raw/ai/2026-05-09-openclaw-architecture-implementation-part2-webclip.md]
updated: 2026-05-09
---

# OpenClaw 技术架构（下）

**来源：** [踏天, 2026-03-26](https://mp.weixin.qq.com/s/FUJEofqbK7vX-J64UX8Nkg)

## 核心结论

如果说上篇的价值在于给出 `Gateway-centered personal agent system` 的总体拓扑，那下篇真正补上的，是这套系统为什么能长期运行而不只是“看起来很强”。

作者重点展开的不是酷炫能力，而是那些把 agent 系统从 demo 推向可持续运行产品的基础层：

- `sandbox`
- `memory`
- `skills`
- `session`
- `routing`
- `nodes`
- `security`
- `config`

也就是说，这篇更像 OpenClaw 的运行机制手册，而不是功能介绍。

## 这篇最值得记的是“文件即真相 + 索引加速器”

下篇里最有代表性的设计，是记忆系统：

- 长期记忆和短期记忆都落在 Markdown 文件里
- SQLite、向量索引、BM25、MMR、时间衰减只是搜索与排序加速器

这个 framing 很关键，因为它说明 OpenClaw 不是把记忆当成某个神秘黑箱，而是坚持：

- 真相层保持人类可读可编辑
- 机器层负责检索、融合和自动刷新

这和 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] 可以对应起来看。那篇讲通用分层，这篇给出一个更接近产品实现的样本。

## 沙箱层说明它默认承认 agent 会出错

文章对 sandbox 的拆解很细，但真正值得保留的是它背后的态度：

- 工具执行要有明确边界
- 容器作用域、工作区访问和网络权限都要可配置
- 工具策略是多层收缩，而不是一次性放开

这意味着 OpenClaw 并不把模型可靠性当作前提，而是默认：

- 模型会误操作
- 工具会被误用
- 所以运行环境必须先把爆炸半径控制住

这一点和 [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox：面向 AI Agent 的通用沙箱平台]] 的底层逻辑一致，只是这里更贴近个人助手产品的具体落地。

## Skills / Session / Routing 是它的“操作系统层”

下篇另一个重要价值，是把几个常被分开讲的模块放进同一控制体系里：

- `Skills` 负责能力加载和按优先级覆盖
- `Session` 负责身份、投递路由、生命周期和历史隔离
- `Routing` 负责多 agent、多频道、多账户、多线程的确定性分发

它们合在一起，说明 OpenClaw 的复杂度已经不在单个 prompt，而在：

- 如何组织长期运行的 agent 身份
- 如何在多入口世界中保持会话边界
- 如何让不同能力、设备和频道进入同一控制平面

这里和 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]、[[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] 是互补关系。前两者更抽象，这篇是一个完整系统样本。

## Nodes 和安全模型一起定义了它不是多租户 SaaS

文章后半段有个很重要的边界条件：OpenClaw 的安全模型是 `trusted operator`，不是多租户隔离总线。

这意味着：

- `sessionKey` 是路由键，不是授权边界
- 插件默认视作可信代码
- 真正的安全边界更接近 OS 用户 / 主机 / Gateway

这个判断很重要，因为它纠正了很多人对个人 agent 系统的错误期待：OpenClaw 不是企业级多租户平台，而是偏 `one operator, one gateway` 的个人控制系统。

而 `Nodes` 的配对、命令白名单、审批和唤醒机制，则说明它在这个模型下，试图把手机、桌面和远程设备纳入同一个执行网络。

## 我的判断与保留

我认为这篇最值得长期保留的，不是某个配置项，而是它展示了一套 agent 产品真正成熟时会长出的“底层器官”：

- 文件化真相层
- 混合检索记忆层
- 多层收缩的 sandbox
- 会话与路由控制面
- 配对节点网络
- 明确的信任边界与配置治理

和上篇一起看，OpenClaw 更像一个 `personal agent OS`，而不是单一助手。

它的局限也很明显：

- 很多内容偏实现清单式罗列
- 有些模块讲的是设计意图，不是完整的压力测试结果
- 安全模型建立在“可信操作者”假设上，不适合被误读成企业多租户模板

但作为 `OpenClaw how it actually works` 的补全页，它是有价值的。

## 阅读评级

🔴 建议深读 — 如果你想理解一个个人助手型 agent 系统为什么最终一定会长出 `sandbox + memory + session + routing + nodes + config` 这些层，这篇值得细读。它补的是“能跑”和“能长期运转”之间的差别

## 与其他页面的关联

- [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]] — 上篇给总体拓扑，这篇补运行机制与安全/会话/配置细节
- [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox：面向 AI Agent 的通用沙箱平台]] — 一篇是通用隔离平台，一篇是个人助手系统如何消费隔离能力
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 那篇偏通用架构，这篇给出文件优先 + 混合搜索的实现样例
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 那篇偏 coding CLI 控制面，这篇偏 personal agent OS 式控制面
