---
tags: [摘要, ai, agent, tools, mcp, skills]
sources: [raw/ai/2026-05-09-function-calling-mcp-skills-differences-practice-webclip.md]
updated: 2026-05-09
---

# Function Calling、MCP 与 Skills 的边界

**来源：** [沈询, 2026-02-26](https://mp.weixin.qq.com/s/dAnNHayrE49FEl8TcLII2Q)

## 核心结论

这篇文章最值得保留的，不是对三个概念做名词解释，而是给出了一个比较清楚的分层判断：

- `Function Calling` 是底层机制，负责把自然语言意图转成结构化工具调用
- `MCP` 是标准化集成层，负责把外部系统接入成本降下来
- `Skills` 是文字化流程层，负责把 SOP、约束和资源组织成可复用能力包

作者的强观点是：`MCP` 和 `Skills` 都不是脱离 `Function Calling` 独立存在的新能力，而是在其之上的不同工程封装。

## Function Calling 是真正的基础层

这篇把工具调用流程拆得很清楚：

- 模型先看到用户请求和工具描述
- 模型输出结构化 `tool_calls`
- 系统按函数名和参数执行真实工具
- 执行结果再回传给模型继续决策

这个拆法的重要性在于，它提醒你 `Agent` 调工具的关键不是“会不会调 API”，而是 `LLM` 是否能稳定地完成一次 `非结构化需求 -> 结构化调用` 的转换。

所以这篇最该记住的判断是：`Function Calling` 不是一个附属能力，而是后续一切工具生态的协议底座。

## MCP 解决的是集成规模问题

文章对 `MCP` 的定位很直接：当外部系统越来越多时，问题已经不是“能不能调工具”，而是“每接一个系统都要单独写一层胶水代码，成本太高”。

在这个语境里，`MCP` 的价值不是让模型更聪明，而是把：

- 外部数据源
- 认证与调用方式
- 工具接口暴露

尽量收敛到统一协议里，减少一对一集成。

这点和 [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] 的视角一致。那篇把 `MCP` 放在知识基础设施的“外部能力访问层”，这篇进一步强调：它本质上还是围绕 `Function Calling` 搭出的标准化接入面。

## Skills 解决的是流程定义问题

文章认为，`Function Calling` 和 `MCP` 都会遇到另一个现实瓶颈：很多任务不是缺工具，而是缺一套能用文字表达、又能被模型按步骤执行的流程定义方式。

它给 `Skills` 的定义可以压缩成：

- 用 `SKILL.md` 写清楚指令和 SOP
- 用附带资源文件补上下文
- 用脚本承接确定性执行环节

然后由模型在需要时加载相关 skill，并继续通过 `Function Calling` 去读文件、跑脚本、调工具。

这个角度的价值在于，它把 `Skills` 从“提示词包”提升成了一种 `流程资产封装形式`。

## 这篇最有价值的是把三者放进同一张图

按这篇的说法，可以把三者理解成三个不同层次的问题：

- `Function Calling`：模型如何稳定发起一次工具调用
- `MCP`：工具和外部系统如何低成本接入
- `Skills`：复杂任务流程如何不用全部硬编码，而是以文字化 SOP 复用

这个分层比“哪个更先进”这种讨论有用得多，因为它直接对应不同工程痛点。

## 和现有页面的关系

这篇和 [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] 很适合一起看，但强调点不同：

- AgentScope 那篇关注 `Skill` 如何被分层发现和按需加载
- 这篇关注 `Skill` 为什么会出现，以及它和 `Function Calling / MCP` 的边界

它也能补 [[wiki/ai/build-ai-agent-system-with-ai-coding|用 AI Coding 快速打造 Agent 系统]] 里容易被一笔带过的一层：当 Agent 系统工具越来越多时，真正难的是能力组织，而不是再多接一个工具。

## 我的判断与保留

我认为这篇最值得留下的是 `分层 framing`，而不是作者对 `MCP` 和 `Skills` “竞合关系” 的强表态本身。因为从工程实践看，两者经常会同时存在：

- `MCP` 负责接系统
- `Skill` 负责教模型怎么在具体任务里用这些系统

但即便不同意它的竞争性判断，这篇仍然有用，因为它明确指出了三种不同问题：

- 结构化调用
- 标准化集成
- 文字化流程

这能避免把所有能力都混叫成“工具调用”。

## 阅读评级

🟡 值得追问 — 如果你正在设计 agent 工具层，这篇能帮你把 `Function Calling / MCP / Skills` 三者从概念混战里拆开。它的价值主要在框架化理解，而不在某个具体实现细节

## 与其他页面的关联

- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 那篇讲 skill 如何被渐进式发现和加载，这篇讲 skill 为何作为流程封装层出现
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — 那篇把 MCP 放进外部能力访问层，这篇补 MCP 与 Function Calling 的工程边界
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 一篇强调 skill 是团队资产，这篇补 skill 在 agent 工具体系里的定位
- [[wiki/ai/build-ai-agent-system-with-ai-coding|用 AI Coding 快速打造 Agent 系统]] — 应用层系统搭起来后，最终都会回到工具暴露、流程复用和能力组织这三个问题
