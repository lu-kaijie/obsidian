---
title: 企业级 Agent 多智能体架构与选型指南 -- 来自1000+行业应用实践积累
source_url: https://mp.weixin.qq.com/s/_bz8DEgp4Lqt-xTa_lWN0A
saved: 2026-05-09
tags: [ai]
---
刘军 *2026年3月20日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJAHlYToq2PsT0gy2byUsPL8tjPPVGCwZL5OC8b24SF8xzE2V8FZaaHf0pnchX2TNX3ZBkGugZ33Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

本文基于我们服务阿里巴巴多条业务线（淘天、闪购、爱橙、云智能、高德、饿了么、1688、蚂蚁、菜鸟等）、众多社区用户（如友邦、海尔、建设银行等）、超 1000+智能体应用实践经验积累。

本文发表前，我们刚刚发布了框架新版本，Spring AI Alibaba 全面升级对 AgentScope 框架支持，以 AgentScope ReActAgent 为核心，全面支持基于 AgentScope 的多智能体编排。

AgentScope Java 1.0.10 版本：

https://github.com/agentscope-ai/agentscope-java/tree/main/agentscope-examplesSpring

AI Alibaba 1.1.2.2 版本：

https://github.com/alibaba/spring-ai-alibaba/releases/tag/v1.1.2.2

背景

## 随着智能体在企业的规模化落地，企业的核心关注点已从「大模型能做什么」转向「如何让 AI 真正驱动业务闭环」。单纯依靠大模型闲聊的时代正在结束，业界焦点越来越多地落在智能体流程自动化——即让多个专职智能体在清晰流程中协作，完成高价值、可复现的任务。业界一些实证研究表明，在处理多跳推理等复杂问题时，多智能体协作架构的准确率比单一基础模型可高出约 32%；通过引入专门的「批评」与「优化」智能体，事实检索准确率可提升约 26%，模型幻觉显著降低。

AgentScope 社区在服务企业用户智能体实践落地的过程中，积累了大量有效的、适用于不同业务场景的多智能体模式，包括 Supervisor、Handoffs、Subagent、Routing 等。在 **AgentScope** **Java** 生态中，我们将这些多智能体模式沉淀为具体的框架抽象与代码示例，用户可根据业务选型直接映射到对应的智能体实现，甚至可以基于我们提供的示例直接修改即可实现业务开发。

从最简单的 ReActAgent 单智能体开始

在构建复杂的 AI 应用时，AgentScope 强调一个务实的工程原则： **单智能体优先（Single Agent First）** 。

```java
publicstaticvoidmain(String[] args){    // 准备工具    Toolkit toolkit = new Toolkit();    toolkit.registerTool(new SimpleTools());
    // 创建智能体    ReActAgent jarvis = ReActAgent.builder()        .name("Jarvis")        .sysPrompt("你是一个名为 Jarvis 的助手")        .model(DashScopeChatModel.builder()            .apiKey(System.getenv("DASHSCOPE_API_KEY"))            .modelName("qwen3-max")            .build())        .toolkit(toolkit)        .build();
    // 发送消息    Msg msg = Msg.builder()        .textContent("你好！Jarvis，现在几点了？")        .build();
    Msg response = jarvis.call(msg).block();    System.out.println(response.getTextContent());}
```

绝大多数日常业务需求，完全可以通过「一个单体大模型 + 一组精准适用的外部工具」来解决。单智能体具有低延迟、易于逻辑推理和方便故障排查的天然优势。只有当业务系统的需求跨越了特定的复杂度阈值时，才应考虑向多智能体架构演进。这些阈值包括：

- **上下文管理（Context Management）** ：任务所需的专业知识体量过大，单个上下文窗口无法承载，需要按步骤或按智能体选择性呈现。
- **职责分工（Division of Labor）** ：不同团队需要独立维护各自领域的专家能力与权限边界，并在清晰边界下组合使用。
- **并行化加速（Parallelization）** ：复杂子任务必须并发执行以大幅降低系统延迟，例如多角度调研、多源查询。
- **结构化流转（Structured Flow）** ：业务要求按照严格的工序（如分类、路由、处理）或角色状态（如销售到客服的切换）进行传递，单智能体难以自然保证这些约束。

在实践中，许多系统采用 **混合级联范式（Agent Cascade）** ：前端通过轻量级模型评估任务复杂度，简单任务由单智能体处理，只有置信度低或高度复杂的请求才下放给多智能体网络。这种级联设计在保持整体准确率的同时，能显著降低部署与算力成本，在数学推理等极端任务中甚至可实现可观的算力削减。因此，建议 **先单智能体 + 工具** ，再根据上述阈值判断是否引入多智能体。

AgentScope 支持的多智能体模式

当前，AgentScope 提供多种开箱即用的多智能体模式，并支持通过 **StateGraph** （基于Spring AI Alibaba AgentScope 生态集成）实现自定义工作流。以下是我们总结的几种常用多智能体模式（包含它们的作用与适用场景）。

| 模式 | 作用 | 适用场景 |
| --- | --- | --- |
| **Pipeline** | 固定流程：顺序（A→B→C）、并行（同一输入给多智能体再合并）、循环（子流程重复直到条件满足） | 流程明确，如自然语言→SQL→评分，或一主题多角度调研→合并报告 |
| **Routing** | 分类 → 专家 → 综合：路由器对输入分类，转发给一个或多个专家，结果合并为单一回答 | 多垂直领域（如 GitHub、Notion、Slack），一次请求完成「分类→专家→合并」 |
| **Skills** | 按需披露：智能体只看到技能名/描述，通过 `read_skill` 按需加载完整内容（如 SKILL.md） | 单智能体多种专长，不想一次性把所有领域文本塞进上下文 |
| **Subagents** | 编排智能体通过 Task 工具将工作委托给子智能体；子智能体可用 Markdown 或代码定义，每次调用无状态 | 领域清晰（日历、邮件等），希望单一入口完成路由与结果合并。  - 和Supervisor类似，只是subagent定义方式不同。 - 相比skills可以实现子智能体间上下文隔离。 |
| **Supervisor** | 监督者将专家当工具调用（一专家一工具，如 schedule\_event、manage\_email） | 领域清晰（日历、邮件等），希望单一入口完成路由与结果合并。  - 和Subagents类似，只是subagent定义方式不同 - 相比skills可以实现子智能体间上下文隔离。 |
| **Handoffs** | 状态驱动：工具更新状态变量（如 active\_agent），图根据该变量路由到不同智能体 | 按角色或顺序交接（如销售 ↔ 支持），对话中「当前负责」的智能体会变化 |
| **Custom Workflow** | StateGraph 自定图：顺序、条件、确定性步骤与智能体步骤混合 | 以上模式不适用的情况下，如需要多阶段、显式控制或非 LLM 与 LLM/智能体步骤混合 |

从实现机制上可以这样理解：

- **Pipeline** 通过全局的 `OverAllState` 在顺序（SequentialAgent）、并行（ParallelAgent）、循环（LoopAgent）三种子形态间传递状态；
- **Routing** 作为分发枢纽，利用分类器解析输入意图并将任务送达领域专家，支持简单一次调用或基于 `StateGraph` 的前后处理扩展；
- **Skills** 采用「选择性披露」——主智能体仅保留技能摘要，通过 `read_skill` 按需加载完整 `SKILL.md` ，有效控制上下文膨胀；
- **Subagents** 由编排者通过 Task 工具委派给在隔离上下文中运行的无状态子智能体；
- **Supervisor** 则将专家视为对话中的工具，由监督者在动态上下文中决定唤醒谁；
- **Handoffs** 通过工具调用更新 `active_agent` 等状态，依赖 `ReplaceStrategy` 与图条件边实现角色间平滑切换；
- **Custom Workflow** 则直接使用 `StateGraph` ，将确定性业务逻辑与 LLM 节点编织成自定义网络。

这些模式既可以单独使用，也可以组合使用——例如 Supervisor 监督者通过 Agent as Tool 调用专家、图中某段用 Handoffs、另一段用 Routing，按流程各部分需求选择最合适的模式即可。

多智能体模式分类：工作流 vs 对话

在上一节提到的所有多智能体模式，总体上可划分为两大类： **工作流模式（Workflow Patterns）与对话模式（Conversation Patterns）** 。企业开发者需要在这二者之间做出权衡，并理解各自的优势与局限。

**工作流自动化（Workflow Automation）**

工作流模式代表系统工程中的 **确定性骨架** ：智能体被编排在预先定义好的有向无环图（DAG）或线性管道中，流程在智能体或节点之间流转，拓扑和状态在图或管道中显式定义。AgentScope 中的 **Pipeline** 、 **Routing** 、 **Handoffs** 、 **Custom Workflow** 均属此类。

- **优势** ：具有极高的可预测性、低资源消耗和良好的审计追踪能力。由于执行路径固定，非常容易调试，是金融审批、文档生成等高合规要求场景的常见选择。
- **劣势** ：要求在开发初期对任务进行较完整的规约，对未曾设想的新型模糊输入适应性有限。

**对话模式 / 自治智能体（Conversational Agents）**

决策过程发生在一个 **连续的对话上下文** 中，由大模型自主决定何时调用何种外部工具。通常只有主智能体与用户交互并将结果输出给用户。 **Supervisor** 、 **Subagents** 、 **Skills** 属于对话模式。

- **优势** ：能够优雅地应对复杂的边缘场景和开放式问题，在人类无法预判路径的环境中展现出较高的适应性。
- **劣势** ：容易产生难以控制的「复合误差（Compounding Errors）」，计算 Token 成本较高，且非确定性导致系统调试相对困难。

**最佳实践：混合工作流**

生产环境的常见做法是采用 **混合工作流** ——以确定的工作流作为应用的「脊椎」，仅在需要高度认知灵活性的特定节点（如意图分类、复杂草案生成）引入自治智能体，从而兼顾系统的可靠性与 AI 的智能弹性。其余能力（如 MsgHub、Multi-Agent Debate）可与上述两类组合使用，用于实现交接、辩论或「智能体即工具」等能力。

| 比较维度 | 工作流模式（Workflow） | 对话模式（Conversational） |
| --- | --- | --- |
| **控制流机制** | 显式的图拓扑或线性管道，可以是路径固定、也可以是基于意图的动态职责交接 | 在连续对话上下文中完全由模型动态决策 |
| **核心优势** | 高重复性、审计友好、极易调试 | 适应开放式任务与未知输入 |
| **关键局限** | 缺乏灵活性，需提前指定步骤或可能的流转方向 | 易产生复合误差，Token 成本较高 |

核心模式详解

本节对其中五种最常用、最易混淆的模式做进一步展开，便于你在实现与选型时快速对标。

**Pipeline：顺序、并行与循环**

Pipeline 为任务执行提供 **序列与并发保证** ，是工作流模式的典型代表。

基于 Spring AI Alibaba 的 **SequentialAgent** 、 **ParallelAgent** 、 **LoopAgent** 与 **AgentScopeAgent** ，子智能体按固定拓扑执行，系统状态通过全局的 **OverAllState** 对象无缝传递，通过 `instruction` 与 `outputKey` 串联各环节的输入输出。

**典型场景** ：

- **顺序** ：自然语言 → SQL 生成器 → SQL 评分器，前一环节输出作为下一环节输入。
- **并行** ：同一主题从技术、金融、市场三个角度同时调研，再合并为一份报告。
- **循环** ：生成 SQL 并评分，若得分低于阈值则迭代优化，直到满足条件或达到最大轮数。

**实现要点** ：

每个子环节用 ReActAgent 构建，再通过

`AgentScopeAgent.fromBuilder(...).instruction(...).outputKey(...)` 封装为管道节点；PipelineService 对外提供 `runSequential` 、 `runParallel` 、 `runLoop` ，业务侧只需传入输入字符串即可获得结构化结果。

示例（顺序管道中定义 SQL 生成器与评分器并组装）：

```java
AgentScopeAgent sqlGenerateAgent = AgentScopeAgent.fromBuilder(sqlGenBuilder)    .instruction("{input}")    .outputKey("sql")    .build();AgentScopeAgent sqlRatingAgent = AgentScopeAgent.fromBuilder(sqlRaterBuilder)    .instruction("Generated SQL: {sql}. User request: {input}.")    .outputKey("score")    .build();
SequentialAgent sequentialAgent = SequentialAgent.builder()    .subAgents(List.of(sqlGenerateAgent, sqlRatingAgent))    .build();
sequentialAgent.invoke(Map.of("input", "帮我生成查询用户统计的SQL"));
```

**Routing：分类 → 专家 → 汇总结果**

在 Routing 模式中， **路由器** 作为 **分发枢纽** ，对输入进行意图解析与分类，将子查询送达至特定领域专家（可并行调用），再将专家返回的结果合并为单一回答。适用于存在多个垂直领域的场景——例如 GitHub、Notion、Slack 各自有独立知识与对应智能体。

**目前在 AgentScope 中，我们提供了两种实现方式** ：

- **Simple** ：业务只调 `RouterService.run(query)` ，内部通过 **AgentScopeRoutingAgent** 一次 `invoke` 完成分类、并行专家调用与框架内 merge；若需最终合成，可由 RouterService 再调用一次 LLM 将各专家结果整合成一条回复。
- **Graph** ：使用 **StateGraph** ，流程为 preprocess → routing 子图 → postprocess。预处理做校验与规范化（如 traceId、截断长度），后处理做最终格式与日志；路由子图仍为「分类 → 并行专家 → merge」，但整条链路状态一致，便于扩展与观测。

**适用** ：输入类别清晰、希望一次请求完成「分类 → 专家 → 合并」时选用 Routing。

**Skills：渐进式披露**

Skills 模式采用「 **选择性披露（Selective Disclosure）** 」机制：让一个智能体具备多种专长，但 **不一次性** 把所有领域文本塞进上下文，从而有效防止上下文窗口污染。智能体在系统提示中只看到技能的 **名称与描述** ；当用户问题涉及某领域时，通过工具 **read\_skill(skill\_name)** 按需加载完整的 SKILL.md 内容，再基于该内容作答。

**实现要点** ：技能以目录形式存放在 classpath（如 `skills/sales_analytics/SKILL.md` ），使用 YAML frontmatter 声明 `name` 、 `description` ，正文为 schema、业务逻辑、示例等。 **ClasspathSkillRepository** 加载这些技能， **SkillBox** 为智能体注入技能系统提示与 `read_skill` 工具；ReActAgent 搭配 DashScopeChatModel、SkillBox 与 InMemoryMemory 即可。

SQL 助手示例中，智能体会根据问题决定调用

`read_skill("sales_analytics")` 或 `read_skill("inventory_management")` ，再生成 SQL。

**适用** ：单智能体多种专长、按需加载、不要求上下文隔离的场景。

**Handoffs：状态驱动的智能体交接**

Handoffs 实现 **状态接管** ： **当前负责对话的智能体** 会随流程动态变化。每个智能体可注册 **交接工具** （如 `transfer_to_support` 、 `transfer_to_sales` ），调用时更新图状态中的变量（如 `active_agent` ）；图在节点完成后根据该状态走 **条件边** ，路由到另一智能体或结束，实现不同职能角色间的平滑切换（如销售转客服）。用户始终只与「当前前台」对话，而前台身份由工具调用决定，适合客服、销售等需要按角色或按顺序交接的场景。

**典型场景** ：销售智能体与支持智能体并存——客户问价格时由销售处理，问技术故障时通过 `transfer_to_support` 转给支持；支持在处理完后若客户要下单，再通过 `transfer_to_sales` 转回销售。状态在多次对话轮次间保持，实现「谁负责当前轮」的显式切换。

**实现要点** ：各智能体以 **AgentScopeAgent** （ReActAgent + Toolkit）作为图的节点；交接工具为普通 `@Tool` ，在工具内通过 `ToolContextHelper.getStateForUpdate(toolContext)` 写入 `active_agent` 等键，图需为该键配置 **ReplaceStrategy** 以便合并更新。每个节点出口配置条件边：根据 `active_agent` 决定下一节点或 `END` 。示例见 `agentscope-examples/multiagent-patterns/handoffs` （销售/支持 + 交接工具 + RouteInitialAction、RouteAfterSalesAction、RouteAfterSupportAction）。

**适用** ：需要按角色或按顺序交接、且用户每次只与一个「当前负责」的智能体对话、该负责方可随工具调用切换时，选用 Handoffs。

**Subagents 与 Supervisor：中心编排与「专家即工具」**

两种模式都是「一个中心智能体协调多个专家」

两种模式的主要区别如下：

- **Subagents** ：编排者（Orchestrator）拆解目标，通过 **Task** 工具（及可选的 **TaskOutput** 查后台任务）委派给无状态的子智能体。调用 Task 时传入 `subagent_type` （如 codebase-explorer、web-researcher）和任务描述；子智能体在 **完全独立的隔离上下文** 中运行，防止指令集重叠导致的交叉污染，再将结果作为工具返回值交给编排智能体。子智能体可用 **Markdown 文件** （YAML frontmatter 定义 name、description、tools）或 **Java API** 定义，每次调用无状态，适合多领域、一个协调者、子智能体无需直接对用户说话的场景。
- **Supervisor** ：监督者将各个领域的专家视为对话中的 **工具** （如 `schedule_event` 、 `manage_email` ），在动态演进的对话上下文中自主决定唤醒哪位专家。专家在限定上下文中执行并返回结果，仅监督者的回复会呈现给用户。对于每个专家子智能体，可通过 `includeContents` 、 `returnReasoningContent` 等参数控制是否传入父流程中的上下文、是否返回当前专家推理过程，以实现灵活的上下文隔离目标。适合领域清晰、专家数量相对稳定、希望单一入口完成路由与合并的场景。

**与 Routing、Skills 的对比** ：Routing 是「预处理式」的独立分类步骤，不维护对话历史；Supervisor 是「对话中」由主智能体根据演进中的上下文动态决定调用谁。Skills 是上下文共享、按需加载技能文本；Subagents/Supervisor 是独立执行、结果汇总，上下文隔离，便于做权限与工具限制。

架构选型指南：如何在模式间做出抉择？

在实际业务中，相似的模式往往让人难以抉择。选型时，先想清楚你更需要的是「固定流程」「一次分类合并」「按角色交接」，还是「单智能体多专长」「编排/监督」「辩论/自定义图」。下面基于 AgentScope 提供明确的选型逻辑与对照表。

**快速选型速查**

| **若你需要…** | **可考虑** |
| --- | --- |
| 固定流水线（顺序、并行或循环） | Pipeline |
| 一次分类后交给专家并合并结果 | Routing |
| 通过工具在智能体间切换（如销售 ↔ 支持） | Handoffs |
| 一个智能体多种专长、按需加载上下文 | Skills |
| 一个编排智能体通过 Task 分发给多个子智能体 | Subagents |
| 一个监督者，每个专家一个工具（如日历、邮件） | Supervisor |
| 自定义图（确定性 + 智能体步骤、多阶段） | Custom Workflow |

**Routing vs Supervisor：怎么选？**

两者都能把工作分发给多个智能体，区别在于 **路由决策的方式** 与 **是否保留对话记忆** ：

- **Routing** ：有独立的 **路由步骤** （如 LLM 分类或规则），对当前输入分类后分发给专家；路由器不维护对话历史，本质是 **预处理** 。若业务只需要 **一次性** 的意图分类，合并结果后 **不需要** 保留各专家的历史交互记忆，应使用 **Routing** 。适合输入类别清晰、一次请求完成「分类 → 专家 → 合并」。
- **Supervisor** ：由 **主监督者** 在持续对话中 **动态决定** 调用哪个专家（以工具形式）；主智能体维护上下文，可多轮多次调用不同专家。若需要进行 **多轮对话的动态编排** ，且主智能体需要根据 **不断演进的对话上下文** 来决定下一步调用谁，则必须使用 **Supervisor** 。

**建议** ：输入类别清晰、一次完成分类与合并用 **Routing** ；需要多轮对话、由主智能体根据上下文灵活调度用 **Supervisor** 。

**Skills vs Subagents / Supervisor：怎么选？**

主要区别在于 **上下文是否隔离** 以及 **安全与权限诉求** ：

- **Skills** ：技能内容通过 `read_skill` **按需加载到主智能体的上下文中** ，与主智能体共享同一段对话上下文。当你希望将多种专长汇聚在 **同一个上下文** 中让模型 **全局统筹** 时，使用 **Skills** 。适合「一个智能体多种专长、按需加载」、不要求隔离的场景。
- **Subagents / Supervisor** ：子智能体或专家在 **独立调用/会话** 中执行，与主智能体上下文 **隔离** ，结果汇总回主智能体。若面临 **严格的安全要求** 、必须对不同专家的工具权限进行 **物理级隔离** ，或为了 **防止提示词交叉干扰（Context Contamination）** ，则应使用具有独立运行上下文的 **Subagents** 或 **Supervisor** 。

**建议** ：希望在一个对话里按需加载多领域知识且不介意上下文共享，用 **Skills** ；需要专职子智能体在隔离上下文中执行再汇总，用 **Subagents** 或 **Supervisor** 。

实现与生态：SAA Graph 编排带来了哪些额外能力

**Spring AI Alibaba 与 AgentScope 的定位与协同**

开源社区中逐渐形成两种不同的智能体应用架构取向：一种以 **Spring AI Alibaba** 为代表，以 **Graph 为核心** 的应用框架，强调工作流编排在 AI 应用开发中的重要性；另一种以 **AgentScope** 为代表，以 **Agentic 为核心** 的应用框架，最大化利用基础大模型的能力（ReActAgent、Memory、Context Engineering 等）。这两种取向都会是企业的主流选择，因此 Spring AI Alibaba 在底层全面支持 AgentScope，通过 **AgentScope Starter** 、 **AgentScope Runtime Starter** 实现 AgentScope 与 Spring 生态的集成，让开发者可以按场景选型： **以 Agentic 为核心** 的 AI 应用推荐使用 AgentScope-Java， **以 Workflow 编排为核心** 的 AI 应用推荐使用 Spring AI Alibaba；二者结合时，即可在 Graph 中编排由 AgentScope 开发的智能体，将多个智能体与普通业务逻辑统一纳入同一套工作流。

Spring AI Alibaba 社区发布的 1.1.2.2 正式版本中，一项重要更新便是对 AgentScope 的编排支持：在 **Spring AI Alibaba Graph** 中可以直接编排由 AgentScope 开发的智能体（如基于 ReActAgent、Model、Toolkit、Memory 构建的 AgentScopeAgent），实现「工作流式」多智能体与确定性业务节点的混合编排。下文从 Graph 引擎为 AgentScope 生态带来的能力角度做简要展开。

**Graph 引擎为多智能体提供的核心能力**

前文将多智能体模式归纳为「对话式」与「工作流式」两类； **Spring AI Alibaba Graph** 主要为 AgentScope 生态提供了编排「工作流式」多智能体的能力，并为长周期、有状态、可观测的智能体流程提供底层支撑。与仅靠单点调用智能体相比，基于 Graph 的编排能带来以下核心能力（可类比业界编排框架所强调的 持久化执行、状态与可观测性 等特性）：

**1\. 统一的编排 API**  
Graph 提供一致的 **StateGraph** / **CompiledGraph** 抽象：节点可以是 AgentScopeAgent、普通函数或子图，边可以是固定边或条件边。开发者通过同一套 API 定义拓扑、编译图并调用，无需为每种模式手写调度逻辑，从而降低「多智能体 + 业务步骤」混合流程的开发与维护成本。

**2\. 流程中的实时状态记录**  
图执行过程中的状态（如 `OverAllState` 及各键上的消息、中间结果）由框架统一维护与传递。每个节点读写共享状态，便于审计、回放与问题定位；同时为「从断点恢复」「跨节点回溯」等能力奠定基础，适合需要 **持久化执行（Durable Execution）** 的长周期任务。

**3\. 标准的状态传递机制**  
通过 **KeyStrategy** （如 ReplaceStrategy、AppendStrategy）为每个状态键定义合并策略，避免各节点自行约定字段格式导致的耦合。前一节点的输出按约定写入指定键，后续节点从状态中按键读取，形成清晰的数据流，便于扩展新节点或调整顺序而不破坏既有契约。

**4\. 适合长周期与可恢复任务**  
工作流式编排天然支持多阶段、长时间运行的任务（如多轮调研、分步审批、迭代优化）。状态由框架管理后，可与持久化存储结合，在故障或重启后从上一检查点恢复，而不是重新跑完全流程，这对企业级可靠性与成本控制尤为重要。

**5\. 对并行场景的原生支持**  
Graph 层支持将同一输入分发给多个节点并行执行（例如 **ParallelAgent** 、Routing 中的多专家并行），再通过 **MergeStrategy** 等机制汇总结果。相比在应用层手写并发与合并逻辑，编排层原生支持能减少重复代码并统一超时、取消等策略。

**6\. 流式与可观测支持**  
执行过程可挂接流式回调与追踪（如 Spring AI Alibaba 与可观测设施的集成），便于将 token 级或节点级输出实时推送到前端，并记录执行路径、状态变迁与耗时，为调试、评估与运维提供可见性，与业界「可观测的智能体运行时」方向一致。

**Spring AI Alibaba Graph** 与 **AgentScope** 的协同，使开发者既能用 AgentScope 的 ReActAgent、Memory、Toolkit 等能力构建高质量的单智能体与多智能体组件，又能用 Graph 的 StateGraph、状态策略与编排 API 将这些组件与业务逻辑编排成可维护、可观测、适合长周期与并行的企业级工作流，形成「Agentic 能力 + Workflow 编排」的完整方案。

体验官方示例

针对本文提到的几种多智能体模式，我们在 AgentScope Java 官方仓库都提供了对应的示例实现，包括 SQL生成、RAG检索、客户服务等贴近企业实践的具体场景示例。

示例源码请参考：

https://github.com/agentscope-ai/agentscope-java/tree/main/agentscope-examples

| 模式 | 实现要点 | 示例路径 |
| --- | --- | --- |
| Pipeline | SequentialAgent / ParallelAgent / LoopAgent + AgentScopeAgent | `agentscope-examples/multiagent-patterns/pipeline` |
| Routing | AgentScopeRoutingAgent（Simple）或 StateGraph + 路由子图（Graph） | `agentscope-examples/multiagent-patterns/routing` |
| Skills | SkillBox + ClasspathSkillRepository + read\_skill | `agentscope-examples/multiagent-patterns/skills` |
| Subagents | TaskToolsBuilder + Markdown/API 子智能体 | `agentscope-examples/multiagent-patterns/subagent` |
| Supervisor | Toolkit.registration().subAgent() | `agentscope-examples/multiagent-patterns/supervisor` |
| Handoffs | StateGraph + 交接工具（更新 active\_agent）+ 条件边 | `agentscope-examples/multiagent-patterns/handoffs` |
| Custom Workflow | StateGraph + 自定节点与边（如 RAG、SQL 工作流） | `agentscope-examples/multiagent-patterns/workflow` |

进入每个示例目录后，均可通过如下命令运行体验（需配置 DashScope API Key 等）：

```nginx
mvn spring-boot:run
```

小结与下一步

AgentScope Java 提供 **Pipeline、Routing、Skills、Subagents、Supervisor、Handoffs、Multi-Agent Debate** 与 **Custom Workflow** 多种模式，你并不需要所有这些模式，它们只是提供了一些通用的参考，找到适合自己业务场景的设计模式，如果需要的话可组合使用。

对于绝大多数智能体场景，我们建议从最简单的 ReActAgent + Tools 的模式开始，在需要的时候引入多智能体架构方案。遇到标准模式无法表达的复杂流程时，再考虑自定义工作流或组合多种模式。

参考资料：本文部分图片、模式参考自 Langchain、Anthropic 等社区项目

大模型 · 目录

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
