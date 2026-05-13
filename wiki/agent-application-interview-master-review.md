---
tags: [分析, ai, interview, agents, llm, python, review]
sources:
  - wiki/agent-runtime-architecture-themes.md
  - wiki/context-knowledge-harness-themes.md
  - wiki/eval-reliability-security-themes.md
  - wiki/enterprise-platform-shifts-themes.md
  - wiki/agent-llm-application-interview-handbook.md
updated: 2026-05-13
---

# Agent 应用开发面试长篇总复习册

## 任务说明

这份文档是在四篇主题综述和已有面试手册之上，再向前整合一层，目标不是继续总结知识库，而是把这些内容重排成一份真正面向面试场景的总复习册。

这份文档想解决四个问题：

- 如何用一套统一框架理解 `Agent 应用开发 / 大模型应用开发`
- 面试里的高频概念题、架构题、系统设计题应该如何组织答案
- 怎样把 `Claude Code`、`OpenClaw`、`Spec / RAG / Harness / Eval / Sandbox` 这些知识点讲成自己的方法论
- 当项目经历不够完整时，如何依然表现出系统理解、工程判断和知识广度

阅读方式建议：

- 先看 `总纲` 和 `十大判断`
- 再看与你最弱的章节
- 面试前最后一轮复习时，重点看 `高频短答`、`系统设计模板`、`开放题观点`

## 总纲：这份总复习册到底在讲什么

如果要把整套知识库和这份面试材料压成一句话，我会这样说：

`Agent 应用开发，不是把模型接进应用，而是围绕一个不确定性的能力核心，设计一套可执行、可恢复、可验证、可治理的任务系统。`

这个定义里有几个关键词：

- `不确定性能力核心`
- `任务系统`
- `可执行`
- `可恢复`
- `可验证`
- `可治理`

也就是说，真正的大模型应用开发，并不只是：

- 调 API
- 写 prompt
- 接向量库

而是把模型、上下文、工具、状态、记忆、评测、安全、工作流和权限这些层组织起来。

这也是为什么这个知识库里虽然主题很多，但最后不断收敛到几条主线：

- Agent 的本质是什么
- Context 为什么比 Prompt 更重要
- RAG、Memory、State 怎么分
- Tool Use、MCP、Skills 怎么分
- Multi-Agent 什么时候该上
- Eval 和 Harness 为什么是生产分水岭
- Claude Code 这类标杆产品真正强在哪里
- 企业和平台层为什么会迅速变厚

## 十大判断：面试时最值得记住的结论

1. 大模型应用开发仍然是软件工程，只是系统中心从确定性逻辑变成了概率型组件。
2. Prompt 很重要，但真正决定系统上限的通常是 `context engineering`。
3. Agent 的本质不是“会聊天”，而是“围绕目标持续执行并闭环”。
4. `Chat`、`RAG`、`Workflow`、`Tool Use`、`Agent`、`Multi-Agent` 必须分开理解。
5. 默认立场应该是 `Single-Agent First`，多 Agent 是复杂度升级，不是默认答案。
6. `RAG` 解决知识补充，`State` 解决任务推进，`Memory` 解决跨轮复用，它们不是一回事。
7. 真正让 demo 变成 production 的不是更强模型，而是 `eval + harness + sandbox + observability`。
8. `Claude Code` 这类标杆产品强的不是界面，而是 `runtime`。
9. AI 时代应用层正在从厚业务逻辑转向厚控制逻辑，也就是上下文、状态、工具编排、验证和治理。
10. 面试里最能拉开差距的，不是背术语，而是把这些判断落到 Python 工程实现和系统设计里。

## 第一章：什么是大模型应用开发

### 1.1 标准定义

大模型应用开发，是把 LLM 作为系统中的能力核心，接入真实业务流程，并通过上下文组织、工具调用、状态管理、评测和安全机制，让它在可控条件下稳定工作。

传统后端的主要对象是确定性规则。  
大模型应用开发的主要对象，是一个：

- 有能力但不稳定
- 能推理但会幻觉
- 能执行但可能越权
- 能生成但不天然可验证

的组件。

所以这类岗位的重点往往不是训练模型，而是设计系统去约束、放大和治理模型能力。

### 1.2 它和传统应用开发的相同点

- 仍然需要分层设计
- 仍然需要状态管理
- 仍然需要测试与监控
- 仍然需要权限和审计
- 仍然需要工程取舍

### 1.3 它和传统应用开发的不同点

- 输出不是确定性的
- 模型表现高度依赖上下文质量
- 工具接口要给模型消费，而不只是给人或程序消费
- 评测不能只看功能是否通过，还要看质量是否稳定
- 上线后模型、prompt、retrieval、tool 都可能单独引入回归

### 1.4 面试短答

如果面试官问“你怎么理解大模型应用开发”，可以答：

`我理解它本质上还是软件工程，但系统中心不再是确定性规则，而是一个不确定性的模型组件。所以关键工作不只是接模型，而是组织上下文、状态、工具、评测和安全边界，让模型在真实业务里稳定工作。`

## 第二章：什么是 Agent，什么不是 Agent

### 2.1 六个概念的边界

`Chat`

- 重点是对话交互
- 不一定有执行动作

`RAG`

- 重点是补充知识
- 不一定有计划或外部动作

`Tool Use`

- 重点是模型调用外部能力
- 但不一定有持续闭环

`Workflow`

- 重点是预定义步骤
- 适合高可控、高审计场景

`Agent`

- 重点是围绕目标多步推进
- 会根据中间结果动态调整动作

`Multi-Agent`

- 重点是多个执行单元之间的职责分工、上下文隔离、并行化或角色切换

### 2.2 什么样的系统才值得叫 Agent

我会看这几个条件：

- 是否有明确目标
- 是否能进行多步执行
- 是否能调用外部能力
- 是否能根据中间反馈调整动作
- 是否存在完成或停止条件

### 2.3 为什么很多所谓 Agent 其实只是 workflow

因为很多系统虽然会调工具，但：

- 没有动态决策
- 没有真正的任务循环
- 没有状态推进
- 没有完成判据

这类系统更准确地说，是“带工具的 workflow”。这不是贬义，反而经常是更稳的工程方案。

### 2.4 高频短答

`我不会把所有带工具的大模型系统都叫 Agent。对我来说，Agent 至少要有目标驱动、多步推进、外部动作和根据反馈调整行为的闭环。否则它更像 workflow 或 tool-augmented chat。`

## 第三章：为什么 `Single-Agent First` 是默认立场

### 3.1 为什么默认不该上多 Agent

多 Agent 的想象空间很大，但现实工程里它自带复杂度：

- 状态同步更难
- 错误传播更长
- 调试更麻烦
- 评测更贵
- 权限边界更复杂

而单 Agent 的优势非常朴素：

- 实现简单
- 观测简单
- 回归测试简单
- 出问题更容易定位

### 3.2 什么情况下才该拆成多 Agent

我会看四类阈值：

- `上下文阈值`
  当前单个上下文已经明显过大，混在一起会污染决策

- `职责阈值`
  不同任务需要不同角色、不同工具集或不同规则

- `并行阈值`
  任务天然可拆，并行能显著提升吞吐

- `权限阈值`
  不同操作需要不同安全边界，必须隔离

### 3.3 常见模式怎么区分

`Router`

- 用来做分类和分发

`Planner`

- 用来拆任务步骤

`Orchestrator-Workers`

- 主控负责任务编排，执行者处理子任务

`Supervisor`

- 用来决定该调哪个专家，以及何时收束

`Subagents`

- 真正的上下文隔离和职责隔离执行单元

`Handoffs`

- 基于状态在不同 agent 之间转移当前负责人

### 3.4 高频短答

`我的默认立场是 single-agent first。只有当上下文、职责、并行收益或权限边界明显超过单 Agent 能承受的范围，我才会考虑拆 agent。因为多 Agent 解决的是复杂度问题，但自己也会制造复杂度。`

## 第四章：为什么 `Claude Code` 是理解 Agent 应用开发的标杆

### 4.1 它为什么不是“聊天 IDE”

如果只把 Claude Code 理解成“能写代码的聊天窗口”，就会错过它最有价值的地方。  
它真正强在于把这些东西做成了统一系统：

- Query loop
- tool runtime
- permissions
- workspace context
- subagent
- recovery
- validation

换句话说，它更像一个 `coding agent runtime`，而不是对话产品。

### 4.2 它揭示了什么共性原则

第一，复杂 Agent 一定会长出 `control plane`。  
无论是 REPL、命令系统、hooks、任务列表还是审批流，本质上都是控制平面。

第二，系统必须把 `聊天` 升级成 `状态机`。  
真正重要的不是一轮轮 message，而是：

- 当前任务在哪个阶段
- 已经做过什么
- 下一步该做什么
- 失败后如何恢复

第三，工具和权限必须制度化。  
如果工具调用、审批和验证是散装的，系统迟早散架。

### 4.3 Claude Code 和普通 Copilot 的差别

- 普通 Copilot 更像编辑器增强层
- Claude Code 更像任务执行层

前者主要帮助“在你写代码时给建议”。  
后者则是“代表你执行一整段工程任务”。

### 4.4 面试短答

`我把 Claude Code 看成 Agent runtime 的标杆，而不是聊天式写码工具。它真正值得学的不是某个产品功能，而是它如何把上下文装配、工具调用、权限治理、任务循环和验证闭环统一到一个长期可扩展的执行系统里。`

## 第五章：Prompt、Context、Harness 三层到底怎么分

### 5.1 Prompt 是什么

Prompt 是局部指令层。它负责：

- 角色设定
- 输出格式
- 任务边界
- 风格和语气

它重要，但通常不是系统成败的第一原因。

### 5.2 Context 是什么

Context 是模型每轮实际看到的全部工作集。  
它可能包括：

- system prompt
- 用户消息
- 历史对话
- 检索结果
- 工具结果
- 规则文档
- 状态摘要
- memory recall

真正的 `context engineering`，不是“塞更多信息”，而是：

- 塞什么
- 不塞什么
- 什么时候塞
- 以什么结构塞
- 如何压缩

### 5.3 Harness 是什么

Harness 是运行环境与治理层。  
它负责：

- 预验证
- 权限
- 执行隔离
- 测试
- verify
- 观察和记录
- 回滚与恢复

### 5.4 为什么这三层必须分开

因为很多团队会把所有问题都归因于“prompt 没写好”，但实际上：

- 模型看错材料，是 context 问题
- 模型做错动作，是 tool / permission / harness 问题
- 模型完成不了长任务，是 runtime / state / stopping condition 问题

### 5.5 高频短答

`Prompt 解决“怎么说”，Context 解决“给模型什么信息”，Harness 解决“模型在什么环境里可靠执行”。真正的大模型系统优化，通常不是盯着 prompt 单点改，而是三层一起设计。`

## 第六章：Spec、RAG、Skills、Memory、State 怎么放在同一张图里

### 6.1 Spec

Spec 解决的是硬约束。  
它告诉模型：

- 什么必须成立
- 结构边界是什么
- 完成标准是什么

在 AI Coding 场景里，Spec 是代码之上的控制面。

### 6.2 RAG

RAG 解决的是动态软上下文。  
它负责把：

- 外部文档
- 历史方案
- 说明文档
- 知识库内容

按需召回给模型。

### 6.3 Skills

Skills 解决的是程序性知识复用。  
它比普通 prompt 更像 SOP 资产，适合封装：

- 操作流程
- 资源目录
- 约束说明
- 脚本入口

### 6.4 State

State 解决的是当前任务推进问题。  
比如：

- 当前步骤
- 已完成动作
- 中间产物
- 待办事项

### 6.5 Memory

Memory 解决的是跨轮、跨会话可复用信息。  
例如：

- 用户偏好
- 项目经验
- 历史有效做法
- 可复用事实

### 6.6 一张统一图

如果把它们统一起来，可以这么理解：

- `Spec`：硬约束层
- `RAG`：软知识层
- `Skills`：流程资产层
- `State`：任务推进层
- `Memory`：长期复用层

### 6.7 高频短答

`我不会把 RAG、memory、state 混成一个概念。RAG 主要补知识，state 主要管当前任务进度，memory 主要做跨轮复用。Spec 则是更硬的约束层，skills 则偏流程资产。`

## 第七章：Function Calling、MCP、Tools、Skills 怎么回答最稳

### 7.1 Function Calling

Function Calling 是模型发起结构化调用的底座。  
它把自然语言意图转换成可执行参数。

### 7.2 MCP

MCP 是外部能力标准化接入协议。  
它解决的是：

- 多系统接入成本
- 能力暴露一致性
- 工具生态互操作

### 7.3 Tools

Tools 是真实可执行能力接口。  
例如：

- 文件读写
- 命令执行
- 数据查询
- 第三方 API

### 7.4 Skills

Skills 不是工具本身，而是教模型如何组织能力的 SOP 包。

### 7.5 工具设计的工程原则

- 输入要结构化
- 输出要便于模型消费
- 报错要可解释
- 结果不要无节制膨胀
- 能幂等尽量幂等
- 高风险动作要可审批

### 7.6 高频短答

`Function Calling 是结构化调用底座，MCP 是能力接入协议，Tool 是具体执行接口，Skills 是对流程和程序性知识的封装。面试里我会把它们看成不同层，而不是同义词。`

## 第八章：为什么 `Eval` 是生产系统的第一类基础设施

### 8.1 没有 eval 为什么就没有稳定迭代

因为 LLM 系统里变动源太多：

- 模型版本会变
- prompt 会变
- 检索策略会变
- tool 会变
- 数据会变

如果没有评测，你只能靠感觉判断系统是否变好。

### 8.2 至少要有哪几类 eval

`Capability eval`

- 看理论能力上限

`Regression eval`

- 防止新变更把旧能力打坏

`Online eval / A/B`

- 看真实业务效果

`Badcase analysis`

- 用失败样本推动修复

### 8.3 为什么 benchmark 不够

因为 benchmark 经常受：

- 环境噪声
- 工具链差异
- 检索污染
- 任务定义偏差

影响。你自己的系统需要自己的回归样本和业务 grader。

### 8.4 面试短答

`我把 eval 看成大模型系统的一等基础设施。因为没有 regression、badcase 和在线比较能力，模型、prompt、retrieval、tool 每次变动都可能把系统带回“凭感觉开发”的状态。`

## 第九章：为什么 `Sandbox`、权限和事故复盘是 Agent 应用的一部分

### 9.1 Agent 的风险和聊天系统不同

聊天系统主要是内容风险。  
Agent 系统更多是动作风险。

风险动作可能包括：

- 写文件
- 删文件
- 执行命令
- 联网调用
- 操作浏览器
- 修改外部业务系统数据

### 9.2 Sandbox 的作用

Sandbox 不是装饰层，而是：

- 限制副作用半径
- 给更高自主性提供前提
- 提供执行隔离
- 增加审计能力

### 9.3 权限治理的基本原则

- 最小权限
- 风险分级
- 高风险动作人工确认
- 明确的可回放日志
- 执行环境与权限策略联动

### 9.4 为什么要看事故复盘

因为事故复盘会提醒你，系统出问题不一定在模型本身，也可能在：

- routing
- serving
- tool layer
- output pipeline
- infra

这个判断能帮你在面试时显得更像做系统的人，而不是只会调模型的人。

### 9.5 面试短答

`我会把 sandbox、permissions 和 postmortem 当成 Agent 应用的一部分，而不是附属安全话题。因为一旦系统能执行动作，核心问题就变成在什么边界里允许它做什么，以及出问题后怎么隔离、审计和复盘。`

## 第十章：Python 落地应该怎么讲

### 10.1 为什么 Python 是主流

- AI SDK 丰富
- 试验速度快
- Web 服务能力够用
- 和向量库、数据库、队列、模型框架连接顺畅

### 10.2 一套常见分层

`API layer`

- FastAPI
- 鉴权
- 请求入口
- 流式响应

`Orchestrator layer`

- agent loop
- task state
- workflow orchestration

`Tool layer`

- 本地工具
- MCP 接入
- 外部 API

`Knowledge layer`

- RAG
- state
- memory
- summaries

`Storage layer`

- Postgres
- Redis
- 向量库
- 对象存储

`Ops layer`

- trace
- metrics
- cost
- eval
- alerts

### 10.3 常见技术选型

- FastAPI 做服务层
- `asyncio` 做并发
- SSE / WebSocket 做流式输出
- Celery / Arq / RQ 做后台任务
- Redis 做缓存和任务状态
- Postgres 做业务与审计数据
- pgvector 或向量库存检索

### 10.4 面试常见坑点

- 阻塞型 tool 卡死事件循环
- 长任务没有持久化状态
- 流式返回和后台任务割裂
- prompt / tool / model 无版本管理
- badcase 不沉淀
- 工具错误不可恢复

### 10.5 面试短答

`如果让我用 Python 落一个 Agent 应用，我会至少把 API、orchestrator、tool、knowledge、storage、ops 这几层分开。因为大模型系统的问题很少只发生在推理层，往往是调用链、状态和治理层一起出问题。`

## 第十一章：系统设计题怎么答

### 11.1 通用答题骨架

任何 Agent / LLM 系统设计题，我都会按这个顺序答：

1. 先明确业务目标和成功指标
2. 再定义输入输出
3. 再讲核心链路
4. 再讲 context / tool / retrieval / state
5. 再讲异步和失败恢复
6. 再讲 eval 和 observability
7. 再讲 security 和 permissions
8. 最后讲成本、延迟和扩展性

### 11.2 设计企业知识库问答

关键点：

- ingestion pipeline
- chunking + indexing
- retrieval + reranking
- source grounding
- session state
- badcase：召回错、引用错、幻觉补全

### 11.3 设计客服 Agent

关键点：

- FAQ 检索
- CRM / 工单系统调用
- 风险问题升级人工
- 工具权限分层
- 审计和可回放

### 11.4 设计代码助手

关键点：

- workspace 感知
- 文件工具和命令工具
- lint / test / build / verify
- query loop 和停止条件
- sandbox 和审批

### 11.5 设计 research agent

关键点：

- planner / worker
- 多源检索
- 中间结果沉淀
- 摘要与 memory
- 周期性验证和人工干预

## 第十二章：开放题如何讲出“有知识面”

### 12.1 为什么说 Prompt 正在退居二线

因为 prompt 只是文本接口，而系统效果越来越取决于：

- 看到了什么信息
- 能做什么动作
- 处在什么权限边界
- 如何持续验证

也就是 context、tooling 和 harness。

### 12.2 为什么说应用层在变薄，控制层在变厚

因为一部分显式业务规则正在迁移到：

- 上下文组织
- 状态机
- 检索与记忆
- 工具编排
- 审批与评测

传统应用层不会消失，但重心会转向治理与控制。

### 12.3 为什么多 Agent 不是默认答案

因为多 Agent 解决的是某些复杂度，但它自己也制造复杂度。  
默认应该先验证单 Agent 是否足够。

### 12.4 为什么 eval 比追新模型更重要

因为没有 eval，模型升级、prompt 调整、tool 变更之后，你根本无法稳定比较版本。

### 12.5 为什么 Claude Code 像“面向模型的操作系统入口”

因为它不只是对话，而是提供：

- 文件系统入口
- 命令执行入口
- 权限系统
- 会话状态
- 工具扩展
- 验证闭环

模型在其中更像推理内核，而不是完整应用。

## 第十三章：项目经历不够强时，怎么答得不虚

### 13.1 不要假装自己做过超大系统

这是最容易露馅的。  
更好的办法是明确区分：

- 我实际做过什么
- 我系统理解到什么程度
- 如果让我落地，我会怎么拆

### 13.2 可以怎么补位

如果你完整 agent 项目经验还不够强，可以强调：

- 你对系统分层的理解很清楚
- 你能把 Python 后端经验迁移到 agent 系统里
- 你知道生产问题主要发生在哪些层
- 你能讲清楚为什么某些设计更稳

### 13.3 一个稳的表达模板

`我现在的优势不在于我已经做过特别巨大的 Agent 平台，而在于我已经把这类系统拆成了比较清晰的工程层次。我知道大模型应用不是调 API，而是 context、tool、state、eval、sandbox、权限和 runtime 的联合系统。如果让我在 Python 体系里落地，我知道应该先做怎样的最小闭环，以及生产化时风险主要会出现在哪些地方。`

## 第十四章：高频短答速记版

### 1. 什么是 Agent

围绕目标持续执行、可调用外部能力、会根据反馈调整行为的任务系统。

### 2. 多 Agent 一定更好吗

不一定。默认 single-agent first，多 Agent 只在复杂度足够高时才值得。

### 3. RAG 和 memory 的区别

RAG 补外部知识，memory 做跨轮复用，它们目标不同。

### 4. 为什么 context 比 prompt 更重要

因为模型真正看到的是整个工作集，而不是孤立的 prompt。

### 5. Harness 是什么

让非确定性模型嵌入确定性交付链路的控制面。

### 6. Claude Code 为什么值得研究

它是 Agent runtime 的高密度标杆样本。

### 7. 生产化的核心是什么

Eval、badcase、sandbox、permissions、observability、recovery。

### 8. Python 落地要注意什么

分层、异步、状态、工具治理、版本化、badcase 回流。

## 第十五章：最后的面试总结陈述

如果需要用一段较完整的话，把你对 Agent 应用开发的理解讲出来，可以用下面这个版本：

`我对 Agent 应用开发的理解，不是把模型接进业务这么简单，而是围绕一个不确定性的推理核心，设计一套可执行、可恢复、可验证、可治理的系统。这里面我最关注的是 runtime，而不是单点 prompt。进一步往下，就是 context engineering、tools、state、RAG、memory、eval、sandbox 和 permissions 这些层怎么一起工作。我的方法论倾向是 single-agent first，先做最小闭环，再按上下文、职责、并行和权限复杂度逐步升级。像 Claude Code、OpenClaw 这类标杆产品对我最大的启发，是复杂 Agent 最终一定会长出控制平面，而生产系统真正的门槛也会从模型能力转向治理能力。`

## 关联页面

- [[wiki/agent-runtime-architecture-themes|Agent 运行时与架构主题综述]]
- [[wiki/context-knowledge-harness-themes|Context、知识基础设施与 Harness 主题综述]]
- [[wiki/eval-reliability-security-themes|Eval、可靠性与安全治理主题综述]]
- [[wiki/enterprise-platform-shifts-themes|企业落地、平台中间层与方法论转向主题综述]]
- [[wiki/agent-llm-application-interview-handbook|Agent 应用开发 / 大模型应用开发面试八股手册]]
