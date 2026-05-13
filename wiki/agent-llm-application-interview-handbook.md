---
tags: [分析, ai, interview, agents, llm, python]
sources:
  - wiki/ai/claude-code-source-runtime-multi-agent.md
  - wiki/ai/function-calling-mcp-skills-differences-practice.md
  - wiki/ai/effective-context-engineering-for-ai-agents.md
  - wiki/ai/building-effective-ai-agents.md
  - wiki/ai/demystifying-evals-for-ai-agents.md
  - wiki/ai/ai-agent-memory-systems-architecture-practice.md
updated: 2026-05-12
---

# Agent 应用开发 / 大模型应用开发面试八股手册

## 任务说明

这份文档的目标不是罗列知识点，而是把当前知识库里反复出现的主线压缩成一套可用于面试的表达框架。

适用场景：

- Python 为主的 `大模型应用开发` 岗位
- 偏 `Agent 应用开发`、`LLM 工程化`、`AI 产品落地` 的岗位
- 既要能回答高频八股，也要能围绕 `Claude Code` 这类标杆产品展开

使用方式：

- 面试前快速复习时，先看 `核心结论` 和 `高频短答`
- 遇到开放题时，按 `定义 -> 边界 -> 工程意义 -> Python 落地` 的顺序回答
- 遇到系统设计题时，优先调用 `系统设计模板`
- 需要长篇串讲版本时，再看 [[wiki/agent-application-interview-master-review|Agent 应用开发面试长篇总复习册]]

## 这套知识库最终沉淀出的 10 个核心结论

1. 大模型应用开发本质上仍然是软件工程，只是系统中心变成了一个不确定性很强的组件。
2. 真正决定效果的往往不是 prompt，而是 `context engineering`。
3. Agent 的本质不是“会聊天”，而是“在约束下持续执行任务”。
4. `Tool Use`、`RAG`、`Workflow`、`Agent`、`Multi-Agent` 必须分开理解。
5. `Single-Agent First` 是默认务实路线，多 Agent 是复杂度升级，不是默认答案。
6. `RAG`、`Memory`、`State`、`Knowledge Base` 不是一回事。
7. 生产系统和 demo 的分水岭，不在模型本身，而在 `eval`、`harness`、`sandbox`、`observability`。
8. `Claude Code` 这类标杆产品的价值，不在“会写代码”，而在 `runtime + context + tools + permissions + workflow` 的整体设计。
9. 应用层正在从厚业务逻辑转向厚控制逻辑，也就是状态、上下文、工具编排、评测和治理。
10. 面试时最容易拉开差距的，不是背术语，而是能否把这些概念落到 Python 工程实现。

## 一、总论：什么是大模型应用开发

### 标准回答

大模型应用开发，就是把 LLM 作为一个不确定性的能力核心，接入一个可用、可控、可迭代的软件系统。传统软件主要处理确定性逻辑，大模型应用除了正常的软件工程问题，还要处理：

- 非确定性输出
- 上下文装配
- 外部工具调用
- 状态与记忆管理
- 评测与回归
- 安全、权限与审计

所以这个岗位的重点通常不在训练模型，而在于如何把模型组织进真实业务流程。

### 可以往上升一层的说法

我更倾向于把它看成“控制一个聪明但不稳定的组件”的工程学。  
传统后端是在写规则；大模型应用开发是在设计一套让模型、工具、上下文、用户输入和验证机制协同工作的运行时系统。

### Python 落地

Python 在这里的优势主要是：

- AI SDK 和生态成熟
- 服务开发速度快
- 适合快速连接模型、向量库、任务队列和外部系统
- `asyncio`、FastAPI、SSE/WebSocket 足以支撑大多数应用层需求

## 二、Agent 的概念边界

### 高频短答

`Chat` 是单轮或多轮对话。  
`Workflow` 是预定义的步骤编排。  
`RAG` 是检索增强生成。  
`Tool Use` 是模型调用外部能力。  
`Agent` 是围绕目标持续决策和执行的一套闭环。  
`Multi-Agent` 是多个 agent 之间的职责分工与协作。

### 面试标准回答

很多团队会把这些概念混着用，但边界要分清：

- `Chat` 重点是对话体验，不一定执行动作
- `RAG` 重点是补知识，不一定有计划和执行
- `Tool Use` 重点是执行外部动作，但不代表有完整目标闭环
- `Workflow` 重点是预先定义流程，适合结构清晰的任务
- `Agent` 重点是围绕目标，在多步过程中做动态决策
- `Multi-Agent` 则是在更复杂任务下，用多个角色来分工、隔离上下文或并行处理

一个系统能不能叫 agent，我更看这几个点：

- 是否有明确目标
- 是否能多步推进
- 是否能调用外部能力
- 是否能根据中间结果调整后续动作
- 是否有任务完成或停止条件

### 我的判断

很多所谓 Agent，实际上只是“带工具的 workflow”。这不是坏事，反而经常更靠谱。知识库里像 [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]] 反复强调的，就是先从简单的 augmented LLM 和 workflow 出发，再逐步升级自主性。

## 三、Prompt、Context、Harness 三层框架

### 高频短答

`Prompt engineering` 解决“怎么说”。  
`Context engineering` 解决“给模型什么信息”。  
`Harness engineering` 解决“模型在什么环境里可靠执行”。

### 面试标准回答

我会把这三层拆开说：

- Prompt 是局部指令设计，关心语气、格式、角色、输出约束
- Context 是输入给模型的全部工作集，包含系统提示词、历史消息、检索结果、工具结果、状态摘要、规则文档等
- Harness 是运行环境，负责权限、沙箱、验证、回滚、测试、重试、观测、流程控制

知识库里反复出现的结论是：  
真正决定效果上限的，通常不是某条“神 prompt”，而是上下文是否对、是否干净、是否压缩得当、工具返回是否可消费、失败后能否恢复。

### 结合标杆产品怎么说

像 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：从启动到多 Agent 扩展层]]、[[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]] 这种材料给出的共同启发是：

- 强系统不是靠更长提示词堆出来的
- 而是靠完整的上下文装配和运行时治理做出来的

### Python 落地

这三层在工程上往往对应：

- Prompt 层：模板、规则、输出 schema
- Context 层：检索器、状态压缩器、会话摘要器、memory 召回器
- Harness 层：tool runner、sandbox、审批流、test runner、logging、trace、cost monitor

## 四、Tool Use、Function Calling、MCP、Skills

### 高频短答

`Function Calling` 是模型把自然语言意图转换成结构化调用。  
`MCP` 是标准化外部能力接入协议。  
`Skills` 是按需加载的文字化 SOP 或能力包。

### 面试标准回答

我会把它理解成三层：

- `Function Calling` 是调用底座
- `MCP` 是集成协议层
- `Skills` 是流程资产层

基于 [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]，比较稳的说法是：

- Function Calling 解决“模型怎样稳定发起结构化动作”
- MCP 解决“外部系统如何低摩擦接进来”
- Skills 解决“复杂任务流程如何以 SOP 形式复用”

### 工具设计原则

面试里如果问“怎么设计一个 Agent 工具”，可以答这几条：

- 输入参数要清晰、可校验
- 输出尽量结构化，方便模型继续消费
- 幂等操作要尽量幂等
- 错误信息要可解释，不能只返回失败
- 工具结果不能太长，否则会污染上下文
- 高风险工具要有权限分层和人工确认

### Python 落地

通常可以这样组织：

- `tools/` 放本地工具定义
- `schemas/` 放 Pydantic 输入输出模型
- `integrations/` 放 MCP 或第三方 API 对接
- `skills/` 放 SOP 文档、脚本和资源

## 五、RAG、Memory、Knowledge Base、State 的区别

### 高频短答

`RAG` 用来补充外部知识。  
`Memory` 用来保留跨轮或跨会话可复用信息。  
`State` 用来记录当前任务进展。  
`Knowledge Base` 是被检索的知识集合，不等于 memory。

### 面试标准回答

这几个概念最容易被混掉，但功能不同：

- `RAG` 主要解决“模型不知道”的问题
- `State` 主要解决“任务做到哪了”的问题
- `Memory` 主要解决“过去发生过什么，以后还值得带进来”的问题
- `Knowledge Base` 主要解决“哪些外部资料可以被检索”

基于 [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]]，我更倾向于把 memory 分成：

- `working memory`：当前会话工作集
- `session memory`：当前会话阶段性摘要
- `long-term memory`：跨会话偏好、事实、经验

### 我的判断

长期记忆很诱人，但应用开发里要谨慎。很多时候先把这三件事做好，收益更大：

- 检索召回
- 会话状态管理
- 上下文压缩

长期记忆的问题不只是存储，而是：

- 提取什么
- 什么时候更新
- 什么时候遗忘
- 错误记忆怎么修正
- 是否涉及隐私和画像风险

### Python 落地

一个常见组合是：

- Postgres / Redis 存状态
- 向量库存可检索知识
- 单独的 memory store 存长期偏好和经验
- 摘要器把长会话压成短摘要回灌

## 六、Agent 架构与选型

### 高频短答

我的默认立场是 `Single-Agent First`。  
只有当任务复杂度、并行收益、权限隔离或上下文隔离需求足够强时，才升级为多 Agent。

### 面试标准回答

单 Agent 的优势是：

- 系统简单
- 评测简单
- 调试简单
- 状态一致性更容易保证

多 Agent 的常见收益是：

- 职责隔离
- 并行执行
- 上下文压缩
- 权限隔离
- 将复杂任务拆成更稳定的子问题

但多 Agent 的代价也很现实：

- 协议和消息边界复杂
- 错误传播路径更长
- 状态同步更难
- 观测和评测成本更高

### 常见模式

- `Router`：决定把任务分给哪个处理器
- `Planner`：把复杂目标拆成步骤
- `Orchestrator-Workers`：主控调度多个执行者
- `Supervisor`：对子 agent 做高层控制和收敛
- `Handoffs`：不同 agent 之间转交上下文和责任

### 结合知识库的结论

[[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]]、[[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]]、[[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] 这一串内容最终指向的是同一件事：  
`架构升级必须由任务复杂度驱动，不要由概念热度驱动。`

## 七、如何理解 Claude Code 这类标杆产品

### 高频短答

Claude Code 不是“聊天框 + 代码补全”，而是一个模型驱动的任务执行 runtime。

### 面试标准回答

如果让我概括 `Claude Code` 这类产品的本质，我会说它把下面这些能力做成了统一系统：

- 任务循环
- 上下文装配
- 工具调用
- 权限治理
- 工作区状态感知
- 多 agent / subagent 扩展
- 验证和恢复

这也是为什么它和普通 AI IDE 插件的差别很大。后者更像增强编辑器；前者更像带控制面的 agent runtime。

### 为什么它是标杆

基于 [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：从启动到多 Agent 扩展层]] 和相关页面，可以抽出几个关键点：

- Query loop 是显式状态机，不是简单聊天轮次
- REPL 更像控制面，不只是消息展示器
- 工具调用、权限确认、后台任务和扩展能力都进入统一执行链
- 它能承接长任务，是因为有压缩、恢复、验证和任务边界，而不是因为“模型更聪明”

### 你可以怎么往深里讲

如果面试官追问，我会进一步说：

- 这类系统最核心的不是 UI，而是 runtime
- runtime 最核心的不是模型接入，而是状态管理和控制逻辑
- 真正难的是让 agent “持续可靠地干活”，而不是偶尔完成一轮惊艳回答

## 八、Eval、Benchmark、Badcase、A/B

### 高频短答

没有 eval，就没有稳定迭代的大模型应用。

### 面试标准回答

在 LLM/Agent 应用里，我会把 eval 看成第一类基础设施。  
原因很简单：模型输出不稳定，prompt 会变，工具会变，检索会变，数据会变，没有评测就无法判断系统到底变好了还是变坏了。

通常至少要分四类：

- `capability eval`：看系统理论能力上限
- `regression eval`：防止新版本把旧能力打坏
- `online eval / A/B`：验证线上真实效果
- `badcase analysis`：用真实失败样本反向驱动改进

### 评测时看什么

- 任务完成率
- 工具调用成功率
- 检索命中率和引用质量
- 人工偏好得分
- latency、token、cost
- 高风险任务的越权率或错误动作率

### 基于知识库的观点

[[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]、[[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]]、[[wiki/ai/production-prompt-ab-evaluation-engineering-practice|生产级 Prompt 自动化推理评估]] 一直在讲同一条主线：

- benchmark 分数不是全部
- 真正重要的是能否形成可回归、可比较、可定位问题的质量治理闭环

### Python 落地

可以这样做：

- 单独维护 `eval dataset`
- 为每条样本定义输入、期望行为、grader
- CI 里跑离线 regression
- 线上保存 badcase 到数据库或对象存储
- 定期抽取失败样本扩充评测集

## 九、安全、权限、Sandbox

### 高频短答

Agent 的风险不在它“说错话”，而在它“做错事”。

### 面试标准回答

一旦 LLM 能调工具、写文件、执行命令、联网调用、访问企业系统，风险级别就和普通聊天机器人完全不一样。  
因此系统里至少要有：

- 最小权限原则
- 分级授权
- 高风险动作人工确认
- 执行隔离
- 审计日志
- 可回放链路

### Sandbox 解决什么

Sandbox 的主要价值不是增强模型能力，而是缩小错误影响半径。  
比如：

- 限制文件系统范围
- 限制网络权限
- 限制命令白名单
- 限制资源消耗

### 可以怎么结合标杆产品

知识库里关于 `OpenSandbox`、`Claude Code auto mode`、`sandboxing` 的内容，都指向一个共识：  
`越想提高 agent 自主性，越需要更强的权限制度和隔离环境。`

## 十、Python 工程落地框架

### 面试标准回答

如果让我用 Python 落一个 Agent 应用，我通常会把系统拆成这几层：

- `API layer`：FastAPI，处理请求、鉴权、流式返回
- `orchestrator layer`：任务编排、agent loop、状态推进
- `tool layer`：本地工具、第三方 API、MCP 接入
- `retrieval/memory layer`：知识检索、状态、记忆
- `storage layer`：Postgres、Redis、对象存储、向量库
- `eval/ops layer`：日志、trace、cost、评测、报警

### 常见技术栈

- Web 服务：FastAPI
- 并发模型：`asyncio`
- 流式输出：SSE 或 WebSocket
- 任务队列：Celery、RQ、Arq，或者基于消息队列自建
- 缓存和状态：Redis
- 业务数据：Postgres
- 检索：向量库或 pgvector

### 常见坑

- 工具调用阻塞事件循环
- 长任务没有状态持久化
- 流式接口和后台任务断链
- prompt、tool、模型版本没有版本化
- 工具报错信息过于原始，模型无法恢复
- badcase 没有沉淀，问题每周重复出现

## 十一、Demo 到 Production 的分水岭

### 高频短答

Demo 的重点是“能跑通”。  
Production 的重点是“能持续稳定、可观测、可回滚、可治理”。

### 面试标准回答

一个大模型 demo 想变成生产系统，至少要补齐这些能力：

- 明确的状态管理
- 工具失败恢复
- 评测和回归
- trace 和日志
- token / latency / cost 监控
- prompt / tool / model 版本管理
- 安全权限和审计
- 降级和回退策略

### 可以升维的说法

这也是为什么很多人觉得“应用层护城河不强”的判断并不完整。  
真正的门槛往往不在聊天框，而在治理层和控制层。

## 十二、系统设计题模板

### 通用答题框架

拿到题目后，按下面顺序讲：

1. 业务目标和成功指标
2. 输入输出是什么
3. 核心链路怎么走
4. 需要哪些 context、retrieval、tools、state
5. 哪些地方异步，哪些地方同步
6. 如何评测和监控
7. 权限和安全怎么做
8. 成本、延迟、扩展性怎么平衡

### 设计企业知识库问答

重点讲：

- 文档 ingestion
- 索引构建
- 检索召回 + rerank
- 引用来源
- 多轮对话中的状态管理
- badcase：召回错、引用错、幻觉补全

### 设计客服 Agent

重点讲：

- FAQ 检索
- 工单系统 / CRM 工具调用
- 风险分级
- 人工转接
- 高风险决策不自动执行

### 设计代码助手

重点讲：

- workspace 感知
- 文件读写工具
- 测试 / lint / build 工具
- 上下文压缩
- 审批流
- sandbox

### 设计长期运行的 research agent

重点讲：

- planner / worker 拆分
- web / docs / local KB 多源检索
- 中间结果沉淀
- 记忆和状态分层
- 周期性总结
- 失败恢复与人工干预

## 十三、高频面试问答

### 1. 什么情况下应该上 Agent，不该上 Agent？

如果任务需要多步决策、外部工具调用、根据中间结果动态调整，适合上 Agent。  
如果流程固定、步骤清晰、出错代价高，优先 workflow，不要为了“高级感”硬上 agent。

### 2. 多 Agent 一定更好吗？

不一定。多 Agent 的收益来自职责隔离和并行化，但代价是状态同步、调试和评测复杂度显著上升。默认还是 `single-agent first`。

### 3. RAG 的核心难点是什么？

不是接个向量库，而是：

- 文档切分是否破坏语义
- 召回是否准
- rerank 是否有效
- 检索结果如何组织进上下文
- 生成时是否真正引用而不是瞎补

### 4. 你怎么看长期记忆？

它有价值，但默认不应做得太重。很多场景先把状态、检索、摘要做好，收益更直接。长期记忆真正难的是更新、遗忘、纠错和隐私治理。

### 5. Function Calling 和 MCP 的区别？

Function Calling 是模型发起结构化调用的机制；MCP 是把外部能力标准化暴露给模型系统的协议层。

### 6. 你怎么看 Claude Code？

我把它看成 coding agent runtime 的标杆。它说明真正强的系统不是 prompt 很厉害，而是状态机、工具执行、上下文装配、权限控制和验证闭环做得完整。

### 7. hallucination 怎么缓解？

分层看：

- 知识不足，用 RAG
- 动作错误，用工具 schema、权限和确认机制
- 上下文混乱，用压缩和状态管理
- 质量不稳，用 eval 和 badcase 闭环

### 8. 你怎么衡量一个 Agent 版本是否变好了？

至少看：

- 任务完成率
- 回归集通过率
- 关键 badcase 是否修复
- latency / token / cost 是否可接受
- 高风险动作错误率是否下降

## 十四、开放题谈资

### 1. 为什么 prompt engineering 正在退居二线？

因为 prompt 只是一段文本，而生产系统真正难的是把正确的信息、正确的工具结果、正确的状态，在正确的时候送进去。也就是 context engineering。

### 2. 为什么说应用层在变薄，控制层在变厚？

因为大量原本写死在业务代码里的决策，开始交给模型和外部工具完成。应用层的核心不再是穷举规则，而是管理状态、上下文、权限、工具编排和质量治理。

### 3. 为什么 eval 比追新模型更重要？

因为没有稳定 eval，模型升级、prompt 调整、工具变更之后，你甚至不知道系统是变好了还是变坏了。评测是让工程团队有方向感的前提。

### 4. 为什么 Claude Code 类产品像“面向模型的操作系统入口”？

因为它提供的不只是问答，而是一套宿主环境：文件系统、命令执行、权限控制、会话状态、工具扩展、任务调度、验证闭环。模型在里面更像一个推理内核，而不是完整应用本身。

## 十五、面试表达模板

### 自我介绍模板

我目前主要关注的是 Python 方向的大模型应用开发，尤其是 Agent 应用的工程落地。我的理解不是停留在 prompt 或 API 调用层，而是更关注上下文工程、工具调用、状态管理、评测闭环和运行时治理。我对 `Claude Code`、`Agent runtime`、`RAG / Memory / Eval / Harness` 这一类问题有系统性整理，比较擅长从工程视角看一个 LLM 应用怎么从 demo 走向可交付系统。

### 项目经历不够强时怎么说

如果项目经历还没形成非常完整的 agent 产品，也可以这样表达：

- 我已经把这类系统拆成了比较清楚的工程层次
- 我知道高频风险点在哪里
- 我能从 Python 工程角度把 API、tool、RAG、state、eval、ops 接起来
- 我不是只会调用模型，而是能讨论系统为什么稳定、为什么失效、如何治理

### 开放题答题模板

一个稳的答法是：

1. 先下定义
2. 再讲边界
3. 再讲工程上的关键矛盾
4. 最后给出自己的倾向性判断

例如被问“你怎么看多 Agent”，不要先表态，而是先说：

- 它解决什么问题
- 它增加什么复杂度
- 什么时候该用
- 我的默认策略是什么

## 十六、最后的压缩记忆版

如果面试前只剩 5 分钟，可以只记下面这些句子：

- 大模型应用开发的核心不是调 API，而是控制一个不确定性的能力核心。
- Prompt 重要，但真正拉开差距的是 context engineering。
- Agent 的本质不是会聊天，而是能在约束下持续执行任务。
- Workflow、Tool Use、RAG、Agent、Multi-Agent 必须分开理解。
- 默认 `single-agent first`，多 Agent 是升级方案，不是默认答案。
- RAG 解决知识问题，state 解决进度问题，memory 解决跨轮复用问题。
- 生产系统的关键不在模型，而在 eval、harness、sandbox、observability。
- Claude Code 这类产品的价值在于 runtime，而不只是界面。
- 应用层正在变薄，控制层正在变厚。
- Python 工程落地时，要把 API、orchestrator、tool、retrieval、storage、eval 分层想清楚。

## 关联页面

- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：从启动到多 Agent 扩展层]]
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]]
- [[wiki/ai/effective-context-engineering-for-ai-agents|Effective context engineering for AI agents]]
- [[wiki/ai/building-effective-ai-agents|Building Effective AI Agents]]
- [[wiki/ai/demystifying-evals-for-ai-agents|Demystifying evals for AI agents]]
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]]
