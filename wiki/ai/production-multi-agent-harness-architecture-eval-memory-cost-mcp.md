---
tags: [摘要, ai, harness, multi-agent, runtime, evaluation]
sources: [raw/ai/2026-05-13-production-multi-agent-harness-architecture-eval-memory-cost-mcp-webclip.md]
updated: 2026-05-13
---

# 从零设计生产级 Multi-Agent Harness：架构、评估、记忆、成本与 MCP 工具接入全拆解

**来源：** [李伟山, 2026-05-13](https://mp.weixin.qq.com/s/JPhcyDc4JwRmnMQ-76A-FQ)

## 核心结论

这篇文章最值得保留的地方，是它把 `Multi-Agent Harness` 明确定义成 Agent 的运行时底座，而不是“多 prompt 拼盘”：

- Agent 负责局部智能
- Harness 负责全局控制
- 生产问题的核心不在模型更强，而在编排、状态、工具治理、评估和成本控制是否外部化

它和 [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] 的判断完全同向，只是这篇更系统地把控制面拆成模块。

## 文章最强的一点是把控制权边界说清楚了

作者反复强调一个原则：

`让 Agent 出主意，让 Harness 拿决定。`

这意味着下面这些权力不该交给单个 Agent 自己裁决：

- 任务生命周期推进
- 计划能否执行与是否并行
- 路由到哪个 Agent
- 失败后的重试、降级或终止
- `max_steps`、`max_tokens`、`max_duration`、`max_tool_calls` 等硬停止条件

这和 [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]]、[[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]] 可以连起来看：前者强调停止条件外部化，后者强调多 Agent 复杂度门槛，这篇补的是运行时控制权的精确落点。

## Tool Registry 被当成安全边界，而不是工具表

文中对工具治理的处理很成熟。它不是把工具写成函数给模型随便调，而是要求所有工具都进入统一 `Tool Registry`，至少登记：

- 名称与描述
- JSON Schema
- 允许调用的 Agent
- 超时、速率限制与风险等级
- 是否需要人工确认
- 输出结构与审计策略

这个判断很重要，因为它说明工具不是能力增强插件，而是生产资源授权点。它和 [[wiki/ai/introducing-advanced-tool-use-on-the-claude-developer-platform|Claude 平台高级工具调用]]、[[wiki/ai/writing-effective-tools-for-ai-agents-using-ai-agents|Writing effective tools for AI agents—using AI agents]] 是同一条线，只不过这篇把治理层抬得更高。

## 状态与记忆的分层讲得很清楚

这篇很克制地把 `state` 和 `memory` 分开：

- `Working State`：当前步骤临时上下文
- `Session State`：单次会话内共享信息
- `Execution Log`：不可变执行轨迹
- `Episodic Memory`：事件经验
- `Semantic Memory`：稳定知识

它还强调两件经常被忽略的事：

- 检索时机要混合设计：前置少量高置信记忆 + 按需 memory search
- 记忆必须有遗忘机制，否则系统会被噪声和过期信息拖垮

这和 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]]、[[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] 能形成很好的工程互证。

## “不要只看答案，要看轨迹” 是这篇最适合带走的评估结论

作者指出，多 Agent 评估不能只盯最终结果，还要评估：

- 计划轨迹是否合理
- 是否调用了错误或未授权工具
- 是否出现无意义循环
- 中间检索和协作是否可信

这和 [[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]]、[[wiki/ai/demystifying-evals-for-ai-agents|拆解 AI agents 评测]] 形成直接呼应：agent 评估最终都会从“答案判对错”走向“执行轨迹判质量”。

## MCP 在这里被放回了正确位置

文末单独讲 MCP 接入，但它没有把 MCP 讲成“有了就高级”，而是把它当成 Tool Registry 背后的标准化工具接入协议。这点很对。

更准确地说：

- Harness 决定工具治理框架
- MCP 负责标准化接入外部工具
- 二者关系类似“控制面”和“适配协议”

这和 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] 的三层拆分高度一致。

## 我的判断与保留

我认为这篇最值得留下的四个点是：

- 明确把多 Agent 生产化问题收敛到 Harness，而不是继续纠缠 prompt
- 清楚划定 Agent 与 Orchestrator 的决策边界
- 把 Tool Registry、状态分层、轨迹评估和预算控制放进同一张图
- 说明 MCP 只是接入层，不是治理层本身

它的局限是：

- 更偏架构蓝图与治理原则，不是可直接照抄的实现文档
- 例子和图示很多，但具体指标、数据结构和故障恢复策略还需要二次设计

## 阅读评级

🔴 建议深读 — 如果你关心的不是“怎么拼几个 Agent”，而是“多 Agent 为什么一进生产就需要控制面、预算闸门和工具治理”，这篇很值得读

## 与其他页面的关联

- [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] — 那篇给出 Harness 的定义，这篇把定义展开成运行时模块
- [[wiki/ai/effective-harnesses-for-long-running-agents|长程 Agent 的有效 Harness]] — Anthropic 官方页偏长程执行闭环，这篇偏多 Agent 架构总图
- [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]] — 那篇讲何时该拆多 Agent，这篇讲拆完之后控制面该长什么样
- [[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]] — 都在说明评估不能只看最终答案
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 那篇讲协议分层，这篇讲协议接入后如何进入治理框架
