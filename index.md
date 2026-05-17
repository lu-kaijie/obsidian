# Index

Wiki 内容目录，每次 ingest 后由 LLM 更新。

## 概念页

_暂无_

## 实体页

_暂无_

## 来源摘要

- [[wiki/ai/claude-personal-guidance|人们如何向 Claude 寻求个人指导]] — Anthropic 2026 年研究，基于约 100 万条对话抽样分析个人指导模式与谄媚问题
- [[wiki/ai/anthropic-institute-agenda|Anthropic 研究所研究议程]] — TAI 四大研究支柱：经济扩散、威胁与韧性、AI 现实影响、AI 驱动研发
- [[wiki/ai/biomysterybench-claude-bioinformatics|BioMysteryBench：Claude 生物信息学能力评估]] — Claude 在生物信息学任务上接近/超越人类专家，难题上准确率之外更关键的是可靠性仍然脆弱
- [[wiki/ai/2028-global-ai-leadership-scenarios|2028：全球 AI 领导权的两种情景]] — Anthropic 用 compute、distillation 与 adoption 三个变量推演 2028 年中美 AI 竞争，并主张民主国家应锁定 12-24 个月前沿领先
- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] — 用约束、SDD 和多 Agent 协作提升 AI 代码生成的可采纳性
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — OpenAI 的 agent-first 工程实践：短 AGENTS、仓库即记录系统、以约束和反馈回路替代手写代码
- [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] — 从组织设计视角讨论 agent 时代的执行图、管理塌缩、新瓶颈与治理结构
- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 从 Chat、RAG、Workflow 到 Agent 的企业 AI 应用全栈地图，覆盖交付、基础设施与安全
- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — AI 写码提速后，代码评审如何靠 agent、规则收敛和 benchmark 做质量治理
- [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox：面向 AI Agent 的通用沙箱平台]] — 从代码执行、浏览器操作到远程 agent runtime，给 AI 应用提供隔离、调度和网络控制层
- [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] — 用“道法术”框架解释 AI 工程不是推倒重来，而是用确定性工程驾驭不确定性智能
- [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] — 从补全、Agent Coding、Rules 到 SDD 的团队演进复盘，结论是采用混合策略而非教条式 spec-first
- [[wiki/ai/api-router-token-arbitrage|API 中转站与 Token 套利生意]] — 从产业视角观察模型接入中间层：统一 API、支付与合规摩擦如何催生中转平台与灰色套利
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 以 QoderWork 七天上线案例为引子，系统梳理 SDD 的流程、与 vibe coding 的边界，以及它作为 harness 工程中层约束的价值
- [[wiki/ai/team-ai-rd-harness-sdd-evolution|告别“氛围编程”]] — 从团队交付视角说明出码率上升不等于提效，必须把 SDD 和 Harness 接进全链路
- [[wiki/ai/progressive-spec-ai-coding-practice-guide|渐进式 Spec 实战指南]] — 强调按需求复杂度渐进加载 Spec 流程，并区分编排层与执行层 AI 的职责边界
- [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] — 把传统 agent 内循环与 Ralph 的外部强制迭代对照起来，说明“持续工作直到完成”依赖的不是更强推理，而是可验证完成条件与 stop hook 控制
- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 从短期记忆、长期记忆到上下文工程和 Memory-as-a-Service，梳理 agent 记忆层为什么会成为独立基础设施
- [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] — 将 agent 记忆抽象为 Raw Ledger、Derived Views 与 Policy 三件套，并把 memory system 建模成显式的 System 2
- [[wiki/ai/hermes-agent-self-improving-source|Hermes Agent 的 Self-Improving 源码拆解]] — 从短记忆、Skill 自生长和自我修补出发，展示 self-improving agent 的另一条实现路线
- [[wiki/ai/ai-inference-kv-cache-what-we-mean|AI 推理中的 KV Cache]] — 从 Transformer 注意力机制讲到 vLLM、SGLang 和 LMCache，说明 KV Cache 正在从局部优化演化成独立推理基础设施
- [[wiki/ai/production-prompt-ab-evaluation-engineering-practice|生产级 Prompt 自动化推理评估]] — 把 A/B 实验巡检决策固化为 LLM 可执行规则系统，重点在 bad case 复盘、排除条件设计与可解释自动下线判断
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 从工程师角色迁移角度解释文档成为真理源、人机结对节奏、skill 自进化和 SDD 式“改文档再重编译”的工作方式
- [[wiki/ai/ai-native-rd-paradigm-doc-driven-evolution|AI 原生研发范式：文档驱动演进]] — 从上下文腐烂、审查瘫痪和维护断层三类工程失序出发，提出把 SDD 变成团队习惯而非重工具链的轻量 SOP
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 用 skill 把能力目录、完整 SOP、资源文件和工具按层次暴露给 agent，解决多能力系统的上下文过载与碎片化问题
- [[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]] — 从标注文档学习标准，统一评测集生成、自动打分、验收与 badcase 分析，并用识图-推理解耦和置信度筛样本提升可靠性
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — 把 Spec 视为硬约束、RAG 视为软上下文、MCP 视为外部能力接口，构造让 AI 真正“懂项目”的组合方案
- [[wiki/ai/build-ai-agent-system-with-ai-coding|用 AI Coding 快速打造 Agent 系统]] — 以电商场景导购 Agent 为例，展示如何从低代码流程迁移到 LangGraph + Skills + A2A + MCP，并用 AI Coding 加速 DSL 到工程代码的重构
- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 把结构化工具调用、标准化外部集成和文字化流程封装拆成三层，澄清 agent 工具体系里的常见概念混淆
- [[wiki/ai/mini-claude-vibe-coding-rebuild|7 小时 Vibe Coding 一个 Mini-Claude]] — 用最小项目拆出 API、tool loop、session manager、CLI 和 dashboard，适合作为 coding agent 入门样本
- [[wiki/ai/agents-md-practice-guide|AGENTS.md 实践指南]] — 把 AGENTS.md 讲成项目导航地图而非百科全书，并给出 monorepo 与私域源码友好化实践
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 从 Command、Subagent、Skills、Hooks 到 PTC，梳理 AI Coding CLI 如何组织上下文、控制面与远端工作流
- [[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]] — 从动态提示词装配、CLAUDE.md 分层上下文到渐进式压缩与委派协议，解释 Claude Code 的工程增益来自哪里
- [[wiki/ai/claude-code-html-artifacts-over-markdown|Claude Code 团队：为什么开始用 HTML artifact 替代 Markdown]] — 当 Agent 主要生成而人主要阅读时，HTML 可能比 Markdown 更适合作为高信息密度、可交互、可分享的知识工件
- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] — 从启动链路、REPL 控制面、Query Loop 状态机到权限与扩展层，解释 Claude Code 为什么能承接复杂 agent runtime
- [[wiki/ai/agent-principles-architecture-engineering-practice|你不知道的 Agent]] — 将 agent loop、控制模式、上下文分层、评测与 harness 串成一张工程总图
- [[wiki/ai/chat-window-to-multi-agent-console-collaboration-shift|从聊天窗口到多 Agent 控制台]] — 把 AI 编程协作从单线程对话改造成多 agent 调度、Review 驱动和运行时观测的控制台范式
- [[wiki/ai/agent-productivity-paradox-collaboration-bottleneck|Agent 时代的生产力悖论]] — 指出 AI 时代真正的新瓶颈往往不是编码，而是协作结构、信息割裂与发布链路断点
- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 从上下文工程、Skill/Subagent 边界到 WorkflowRepo，说明复杂业务 agent 为什么需要流程级编排和初始化协议
- [[wiki/ai/ai-product-three-year-retrospective-change-and-constants|AI 产品三年复盘：变与不变]] — 从 AI 产品工程师的三年协作演进出发，总结执行方式巨变之下，沟通、判断、责任与信任这些核心能力为何反而更重要
- [[wiki/ai/business-logic-collapse-ai-agent-era-what-to-build|业务逻辑的“坍塌”]] — 解释 AI 时代应用层为何从厚业务代码转向上下文、状态、工具编排与治理控制层
- [[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] — 将 Single Agent、Multi-Agent、Agent Skills、Agent Teams 串成演化路径，并给出由简入繁的选型顺序与升级条件
- [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]] — 以 Gateway 为统一控制平面，拆解 personal agent 系统如何组织 runtime、channels、nodes、skills 与 sandbox
- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — 补全 sandbox、memory、skills、session、routing、nodes、安全与配置层，解释这类 personal agent system 如何长期稳定运行
- [[wiki/ai/openclaw-long-term-memory-pipeline-stability|OpenClaw 长期记忆：优秀管线与玄学效果]] — 拆解日记忆、长期记忆、Dreaming 与 promote 机制，指出 file-first memory 的优点与默认不稳定性
- [[wiki/ai/openclaw-prompt-context-harness-philosophy|OpenClaw 的 Prompt / Context / Harness 设计哲学]] — 用三层框架总结 OpenClaw 如何做动态提示词装配、上下文分层与长期运行环境设计
- [[wiki/ai/openclaw-practice-six-agents-mac|OpenClaw 实战：六个 AI Agent]] — 从真实多 Agent 团队、cron 运营、ACP 委派和记忆自进化出发，展示 personal agent system 如何从能聊进化到能干活
- [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] — 用 Aegis 案例说明 Harness 如何把非确定性模型嵌进确定性交付链路，并重塑工程师的控盘角色
- [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] — 从 Commit Graph、RepoWiki、Memory 到 Agentic Search，具体展开 Harness Engineering 里的工程知识底座应如何组织
- [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] — 从 Prompt/Context/Harness 三段演进解释为什么 AI 主权下放后，环境治理会变成新的工程基本功
- [[wiki/ai/ai-coding-rate-90-harness-engineering|AI Coding 率提升至 90% 的 Harness 实战]] — 用规则、技能、知识库和变更追溯四要素，展示复杂 Java 应用里的 harness 落地方式
- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] — 把 Harness 落到具体仓库结构、预验证、validate 管道和 verify 闭环，强调让 Agent 动手前先验证“能不能做”
- [[wiki/ai/agentic-os-first-agent-oriented-operating-system|阿里云发布 Agentic OS]] — 从平台视角把 skills、shell、token 可观测、安全与 runtime 整体上提到面向 agent 的操作系统层
- [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]] — 将 Pipeline、Routing、Skills、Subagents、Supervisor、Handoffs 等模式映射到企业场景，强调 Single Agent First 与混合工作流
- [[wiki/ai/a-postmortem-of-three-recent-issues|Anthropic：三起近期事故复盘]] — 从上下文路由、输出损坏到 XLA/TPU 误编译，展示 LLM serving 的多层失效样本
- [[wiki/ai/best-practices-for-claude-code|Claude Code 最佳实践]] — Anthropic 官方使用指南，强调探索、计划、执行与验证闭环
- [[wiki/ai/building-a-c-compiler-with-a-team-of-parallel-claudes|并行 Claude 团队构建 C 编译器]] — 用高质量测试、任务切分与并行执行展示多 agent 工程样本
- [[wiki/ai/building-agents-with-the-claude-agent-sdk|用 Claude Agent SDK 构建 Agent]] — 从 agent loop、工具和文件系统出发解释 SDK 如何承载 agent 骨架
- [[wiki/ai/building-effective-ai-agents|构建高效 AI Agents]] — Anthropic 的 agent 方法总论：从 augmented LLM 到 workflow / agent 组合
- [[wiki/ai/claude-code-auto-mode-a-safer-way-to-skip-permissions|Claude Code auto mode]] — 用分类器和固定模板解释“跳过权限”如何仍保持安全边界
- [[wiki/ai/claude-desktop-extensions-one-click-mcp-server-installation-for-claude-desktop|Claude Desktop Extensions]] — 通过一键安装降低 MCP server 接入与分发摩擦
- [[wiki/ai/claude-swe-bench-performance|Claude SWE-Bench 表现]] — Anthropic 官方 coding benchmark 样本，强调 agent 与 tool use 的作用
- [[wiki/ai/code-execution-with-mcp-building-more-efficient-ai-agents|用 MCP 做代码执行]] — 从工具定义和工具结果的 token 成本切入，讨论更高上下文效率的 tool 设计
- [[wiki/ai/contextual-retrieval-in-ai-systems|Contextual Retrieval]] — 用补上下文的检索策略改进传统 RAG 的 chunk 语义缺失问题
- [[wiki/ai/demystifying-evals-for-ai-agents|拆解 AI agents 评测]] — 从任务、grader、capability 与 regression 评测解释为什么 eval 是 agent 基础设施
- [[wiki/ai/designing-ai-resistant-technical-evaluations|设计抗 AI 的技术评估]] — 探讨强模型时代 take-home 技术评估为何失效以及如何重构
- [[wiki/ai/effective-context-engineering-for-ai-agents|高效上下文工程]] — Anthropic 的 context engineering 总论：检索、组织、压缩与长任务上下文维护
- [[wiki/ai/effective-harnesses-for-long-running-agents|长程 Agent 的有效 Harness]] — 从环境管理、增量进展、测试和快速上手构成长程 agent harness 骨架
- [[wiki/ai/equipping-agents-for-the-real-world-with-agent-skills|让 Agent 用上 Skills]] — 解释 Skill 作为按需加载程序性知识单元如何减少上下文负担
- [[wiki/ai/eval-awareness-in-claude-opus-4-6-browsecomp-performance|Claude Opus 4.6 的评测感知问题]] — 讨论 contamination 与多 agent 放大如何污染 benchmark
- [[wiki/ai/harness-design-for-long-running-application-development|长程应用开发的 Harness 设计]] — 把 harness 具体落到 full-stack 应用开发任务和迭代验证闭环
- [[wiki/ai/how-we-built-our-multi-agent-research-system|Anthropic 多 Agent 研究系统]] — 官方研究场景多智能体架构样本，覆盖 prompt、eval 与可靠性
- [[wiki/ai/introducing-advanced-tool-use-on-the-claude-developer-platform|Claude 平台高级工具调用]] — 从 Tool Search Tool 到 Programmatic Tool Calling 讲工具发现与管理层
- [[wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing|Claude Code 沙箱化]] — 用执行隔离换取更高自主性，是 Claude Code 安全边界的一手材料
- [[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|量化 agentic coding eval 的基础设施噪声]] — 说明 benchmark 波动常来自环境噪声而非模型能力变化
- [[wiki/ai/scaling-managed-agents-decoupling-the-brain-from-the-hands|Managed Agents：解耦 brain 与 hands]] — 通过分离高层推理和低层执行来扩展 agent runtime
- [[wiki/ai/the-think-tool-enabling-claude-to-stop-and-think|\"think\" tool]] — 把显式思考停顿外显成运行时工具，而不只是提示词技巧
- [[wiki/ai/writing-effective-tools-for-ai-agents-using-ai-agents|写好 Agent 工具]] — 从工具定义、原型、评测到迭代，系统讨论 tool 设计方法论
- [[wiki/ai/claude-code-skills-source-analysis|Claude Code 的 skills 源码解析]] — 从加载链路、条件激活和 frontmatter 声明解释 skill 为什么是 Claude Code runtime 的一级能力构件
- [[wiki/ai/production-multi-agent-harness-architecture-eval-memory-cost-mcp|生产级 Multi-Agent Harness 全拆解]] — 把多 Agent 生产化问题收敛到控制面、Tool Registry、状态分层、轨迹评估和预算闸门
- [[wiki/ai/llm-wiki-obsidian-wiki-gbrain-knowledge-evolution|LLM Wiki / Obsidian-Wiki / GBrain]] — 用“编译式知识系统”视角解释当前 wiki 这类 raw→wiki→schema 三层知识工程范式

## 综合分析

- [[wiki/知识管理-最佳实践|知识管理最佳实践]] — 如何让个人知识库真正发挥作用，避免只存入不取出

## 个人笔记

- [[1.agent-llm-application-interview-handbook|Agent 应用开发 / 大模型应用开发面试八股手册]] — 基于当前知识库压缩出的面试作战手册，覆盖 Agent、RAG、Memory、Tool Use、Eval、Sandbox、Claude Code 与 Python 工程落地
- [[2.agent-runtime-architecture-themes|Agent 运行时与架构主题综述]] — 压缩全库中关于 workflow、runtime、Claude Code、OpenClaw 与多智能体选型的主线
- [[3.context-knowledge-harness-themes|Context、知识基础设施与 Harness 主题综述]] — 总结 Prompt/Context/Harness、Spec、RAG、Memory、AGENTS.md 与 repo 级知识分层
- [[5.eval-reliability-security-themes|Eval、可靠性与安全治理主题综述]] — 总结评测、badcase、benchmark 噪声、sandbox、权限与生产事故主线
- [[4.enterprise-platform-shifts-themes|企业落地、平台中间层与方法论转向主题综述]] — 总结企业 AI 架构、业务逻辑迁移、平台层膨胀、协作瓶颈与组织转型
- [[6.agent-application-interview-master-review|Agent 应用开发面试长篇总复习册]] — 基于四篇主题综述与面试手册重排出的总复习册，按概念题、架构题、系统设计题与开放题组织
- [[7.genericagent-principles|GenericAgent 原理笔记]] — 围绕 context density、最小原子工具、分层记忆与运行时压缩整理出的结构化阅读笔记
- [[9.TencentDB Agent Memory|TencentDB Agent Memory 项目做法总结]] — 聚焦分层记忆与短期上下文卸载，适合作为 memory 系统的落地案例
- [[10.hermes-agent-memory-and-knowledge|记忆与知识系统 — 实现解析]] — 拆 Hermes 的持久记忆、会话搜索与外部 memory provider 分层
- [[11.hermes-agent-self-improving-learning-loop|自我改进学习循环 — 实现解析]] — 聚焦 skill 创建、后台审查与自我改进闭环
- [[12.rag|RAG 资料笔记]] — 从检索、切分、embedding 到评估，保留一篇偏入门的工作态资料
- [[13.claude-code-cmd|Claude Code 命令流笔记]] — 记录 `/powerup`、`@` 引用和 CLI 使用细节，便于后续整理为操作手册
- [[14.claude-code-source-code-explain|Claude Code 源码讲解资料笔记]] — 收一篇偏源码导读的长文，后续可继续提炼 runtime 结构
- [[15.claude-code-muti-agents|Claude Code 多 Agent 机制资料笔记]] — 聚焦 subagent、fork 与 coordinator 协作模式
- [[16.Harness Engineering|Harness Engineering 资料笔记]] — 保留 Mitchell 语境下“把错误修进环境”的典型论述
- [[17.OpenClaw|OpenClaw 资料笔记]] — 收一篇偏 OpenClaw 架构科普/面试向材料
- [[18.GraphRAG|GraphRAG 资料笔记]] — 保留 GraphRAG / LightRAG 的工作态面试材料

## 特殊页面

- [[wiki/overview|概览]] — 知识库整体状态与主题地图
