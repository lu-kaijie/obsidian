---
tags: [概览]
updated: 2026-05-15
---

# Wiki 概览

知识库整体状态，由 LLM 在每次 Lint 时维护。

## 当前状态

- **主题数**：2（AI、知识管理）
- **Wiki 页面数**：83
- **AI 页面数**：81
- **已摄入资料数**：83
- **最后更新**：2026-05-15

## 主题地图

### AI（81 篇）

#### 1. Harness / Agent-first 工程

- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — agent-first 工程的总样本：仓库即记录系统、短 AGENTS、强约束与自动反馈回路
- [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] — 从 Prompt / Context / Harness 三阶段解释环境治理为什么成为新的工程共识
- [[wiki/ai/qoder-harness-engineering-guide|Qoder Harness Engineering 指南]] — 把 Harness 落成仓库结构、预验证、validate 管道和 verify 闭环
- [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] — 用 Aegis 案例说明 Harness 如何把非确定性模型嵌进确定性交付链路，并区分伪 Harness / 劣质 Harness / 可用 Harness
- [[wiki/ai/ai-coding-rate-90-harness-engineering|AI Coding 率提升至 90% 的 Harness 实战]] — 用规则、技能、知识库和变更追溯四要素，展示复杂 Java 应用里的 harness 落地方式
- [[wiki/ai/effective-harnesses-for-long-running-agents|长程 Agent 的有效 Harness]] — Anthropic 官方一手材料，强调环境管理、增量进展与测试闭环
- [[wiki/ai/harness-design-for-long-running-application-development|长程应用开发的 Harness 设计]] — 将 harness 具体化到 full-stack 应用开发任务
- [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] — 进一步补足 Harness 里的知识底座：Commit Graph、RepoWiki、Memory、Agentic Search

#### 2. Spec / 文档驱动编码

- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 把 Spec 从文档提升为 AI 编码的执行控制层
- [[wiki/ai/progressive-spec-ai-coding-practice-guide|渐进式 Spec 实战指南]] — 强调按需求复杂度渐进加载流程，并区分编排层与执行层 AI
- [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] — 展示团队如何从补全、rules 走向更现实的混合式 SDD
- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 强调文档是源码、代码是编译结果的工作方式迁移
- [[wiki/ai/ai-native-rd-paradigm-doc-driven-evolution|AI 原生研发范式：文档驱动演进]] — 从上下文腐烂、审查瘫痪和维护断层出发，给出轻量门禁式 SOP
- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理]] — 将约束、Spec、多 Agent 协作和多仓工程实践拼成可执行工作流
- [[wiki/ai/claude-code-html-artifacts-over-markdown|Claude Code 团队：为什么开始用 HTML artifact 替代 Markdown]] — 说明当工件主要由 agent 生成、人类主要负责阅读和反馈时，HTML 可以成为比 Markdown 更高信息密度的知识介质
- [[wiki/ai/mini-claude-vibe-coding-rebuild|7 小时 Vibe Coding 一个 Mini-Claude]] — 用一个极简 coding agent 样本解释 API、tool loop、session manager、CLI 和 dashboard 的最小闭环
- [[wiki/ai/team-ai-rd-harness-sdd-evolution|告别“氛围编程”]] — 从团队交付视角说明出码率上升不等于提效，必须把 SDD 和 Harness 接进全链路

#### 3. Skills / Workflow / Agent 编排

- [[wiki/ai/function-calling-mcp-skills-differences-practice|Function Calling、MCP 与 Skills 的边界]] — 结构化工具调用、标准化外部集成和文字化流程封装的三层拆分
- [[wiki/ai/agentscope-skills-progressive-disclosure|AgentScope Skills 与渐进式披露]] — 解决多能力系统上下文过载的 skill 组织方式
- [[wiki/ai/equipping-agents-for-the-real-world-with-agent-skills|让 Agent 用上 Skills]] — Anthropic 对 skill 作为按需加载程序性知识单元的官方说明
- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 从 skill / subagent 边界一路讲到 workflow 级编排
- [[wiki/ai/building-effective-ai-agents|构建高效 AI Agents]] — Anthropic 的 agent 方法总论，强调先 workflow 后 agent 的渐进升级
- [[wiki/ai/building-agents-with-the-claude-agent-sdk|用 Claude Agent SDK 构建 Agents]] — 从 SDK 视角拆解 agent loop、工具注册、状态管理与工程接线方式
- [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]] — 从 Command、Subagent、Skills、Hooks 到 PTC 的 CLI 控制面拆解
- [[wiki/ai/claude-code-prompt-context-harness-practice|Claude Code 的 Prompt / Context / Harness 设计实践]] — 从动态提示词装配、CLAUDE.md 分层上下文到渐进式压缩与委派协议，解释 Claude Code 的工程增益来自哪里
- [[wiki/ai/claude-code-skills-source-analysis|Claude Code 的 skills 源码解析]] — 从加载链路、条件激活和 frontmatter 声明解释 skill 如何被扫描、激活并注入 Claude Code runtime
- [[wiki/ai/best-practices-for-claude-code|Claude Code 最佳实践]] — Anthropic 官方使用指南，强调探索、计划、执行与验证闭环
- [[wiki/ai/the-think-tool-enabling-claude-to-stop-and-think|Think Tool]] — 通过显式“停下来想一想”的工具调用，为复杂任务提供额外推理缓冲层
- [[wiki/ai/agents-md-practice-guide|AGENTS.md 实践指南]] — 把 AGENTS.md 讲成项目导航地图而非百科全书，并给出 monorepo 与私域源码友好化实践
- [[wiki/ai/chat-window-to-multi-agent-console-collaboration-shift|从聊天窗口到多 Agent 控制台]] — 把 AI 编程协作从单线程对话改造成多 agent 调度、Review 驱动和运行时观测的控制台范式
- [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] — 说明“持续工作直到完成”依赖的是外部可验证完成条件
- [[wiki/ai/build-ai-agent-system-with-ai-coding|用 AI Coding 快速打造 Agent 系统]] — 展示从低代码流程迁移到 LangGraph + Skills + A2A + MCP 的工程路径
- [[wiki/ai/agent-principles-architecture-engineering-practice|你不知道的 Agent]] — 将 agent loop、控制模式、上下文分层、评测与 harness 串成一张工程总图
- [[wiki/ai/agent-skills-teams-architecture-evolution-selection|Agent / Skills / Teams 的架构选型]] — 从单智能体一路升级到技能、多智能体和 agent teams 的演进顺序
- [[wiki/ai/enterprise-agent-multi-agent-architecture-selection-guide|企业级多智能体架构选型]] — 将 Pipeline、Routing、Skills、Subagents、Supervisor、Handoffs 映射到企业场景
- [[wiki/ai/how-we-built-our-multi-agent-research-system|Anthropic 多 Agent 研究系统]] — 展示研究型多智能体系统如何分工、调度与合并结果
- [[wiki/ai/production-multi-agent-harness-architecture-eval-memory-cost-mcp|生产级 Multi-Agent Harness 全拆解]] — 从控制面、Tool Registry、状态分层、轨迹评估到预算闸门，系统化拆开多 Agent 生产底座

#### 4. Memory / 上下文 / 知识层

- [[wiki/ai/ai-agent-memory-systems-architecture-practice|AI Agent 记忆系统]] — 记忆基础设施总览：短期 / 长期 / 上下文工程 / Memory-as-a-Service
- [[wiki/ai/memory-architecture-ledger-views-policy|Memory 架构与思考]] — 把 memory 抽象成 Raw Ledger、Derived Views、Policy 三件套，并提出显式 System 2
- [[wiki/ai/openclaw-long-term-memory-pipeline-stability|OpenClaw 长期记忆：优秀管线与玄学效果]] — 拆解 file-first memory 的写入、晋升、Dreaming 与召回链，指出其优雅与不稳定并存
- [[wiki/ai/hermes-agent-self-improving-source|Hermes Agent 的 Self-Improving 源码拆解]] — 从短记忆、Skill 自生长和自我修补出发，展示 self-improving agent 的另一条实现路线
- [[wiki/ai/effective-context-engineering-for-ai-agents|高效上下文工程]] — Anthropic 对 context engineering 的官方总纲
- [[wiki/ai/contextual-retrieval-in-ai-systems|Contextual Retrieval]] — 通过给检索块补上下文来提升 RAG 命中质量
- [[wiki/ai/spec-rag-ai-programmer-knowledge-infra|Spec + RAG 的 AI 程序员知识基础设施]] — 用 Spec、RAG、MCP 组织项目级知识与外部能力
- [[wiki/ai/llm-wiki-obsidian-wiki-gbrain-knowledge-evolution|LLM Wiki / Obsidian-Wiki / GBrain]] — 用“编译式知识系统”视角解释 raw→wiki→schema 的知识工程范式

#### 5. Runtime / OS / 推理与沙箱基础设施

- [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox]] — Agent runtime 的执行隔离层
- [[wiki/ai/claude-code-source-runtime-multi-agent|Claude Code 源码拆解：运行时与多 Agent 扩展层]] — 从启动链路、REPL 控制面、Query Loop 状态机到权限与扩展层，解释成熟 coding agent runtime 的骨架
- [[wiki/ai/scaling-managed-agents-decoupling-the-brain-from-the-hands|Managed Agents：解耦 brain 与 hands]] — 通过分离高层推理和低层执行来扩展 agent runtime
- [[wiki/ai/making-claude-code-more-secure-and-autonomous-with-sandboxing|Claude Code 沙箱化]] — 用执行隔离换取更高自主性
- [[wiki/ai/openclaw-architecture-implementation-part1|OpenClaw 技术架构（上）]] — personal agent OS 的总体控制平面样本
- [[wiki/ai/openclaw-architecture-implementation-part2|OpenClaw 技术架构（下）]] — 补全 sandbox、memory、session、routing、nodes、安全与配置机制
- [[wiki/ai/agentic-os-first-agent-oriented-operating-system|阿里云发布 Agentic OS]] — 从平台视角把 skills、shell、token 可观测与安全能力下沉到 OS 层
- [[wiki/ai/ai-inference-kv-cache-what-we-mean|AI 推理中的 KV Cache]] — 说明 KV Cache 如何从显存优化长成独立控制层与缓存层

#### 6. 评测 / 质量治理 / 决策系统

- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — 代码生成提速后，质量如何靠 rules、agent review 和 benchmark 收敛
- [[wiki/ai/evaluation-agent-unified-automation|评测 Agent 的统一自动化架构]] — 把“学标准、打分、验收、badcase 分析”收敛成统一 workflow agent
- [[wiki/ai/production-prompt-ab-evaluation-engineering-practice|生产级 Prompt 自动化推理评估]] — 通过显式规则、排除条件和 bad case 迭代，把 prompt 做成生产决策程序
- [[wiki/ai/demystifying-evals-for-ai-agents|拆解 AI agents 评测]] — Anthropic 的 eval 总论页，解释 grader、regression 与 capability eval
- [[wiki/ai/claude-swe-bench-performance|Claude 在 SWE-bench 上的表现]] — 用公开基准观察 coding agent 在工程任务上的能力上限与边界
- [[wiki/ai/designing-ai-resistant-technical-evaluations|设计抗 AI 的技术评估]] — 当模型能完成标准题后，技术评估必须转向真实判断与协作能力
- [[wiki/ai/quantifying-infrastructure-noise-in-agentic-coding-evals|量化 agentic coding eval 的基础设施噪声]] — 说明 benchmark 波动有时来自环境而非模型
- [[wiki/ai/eval-awareness-in-claude-opus-4-6-browsecomp-performance|Claude Opus 4.6 的评测感知问题]] — 从 contamination 角度讨论 benchmark 可解释性

#### 7. 组织 / 产品 / 方法论视角

- [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程]] — 用“道法术”框架给 AI 工程做总解释
- [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] — 从 execution graph、Harness / Hive Mind 和治理重构看 agent 时代组织设计
- [[wiki/ai/ai-product-three-year-retrospective-change-and-constants|AI 产品三年复盘：变与不变]] — 说明当“做”变便宜后，“定义 / 判断 / 负责”会更稀缺
- [[wiki/ai/business-logic-collapse-ai-agent-era-what-to-build|业务逻辑的“坍塌”]] — 解释应用层为什么会从厚业务代码转向上下文、状态、工具编排与治理控制层
- [[wiki/ai/agent-productivity-paradox-collaboration-bottleneck|Agent 时代的生产力悖论]] — 指出 AI 时代真正的新瓶颈往往不是编码，而是协作结构、信息割裂与发布链路断点

#### 8. 企业 / 平台 / 产业观察

- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 企业 AI 应用的总架构地图
- [[wiki/ai/claude-desktop-extensions-one-click-mcp-server-installation-for-claude-desktop|Claude Desktop Extensions]] — 把 MCP server 的安装与分发进一步产品化，降低工具接入门槛
- [[wiki/ai/api-router-token-arbitrage|API 中转站与 Token 套利生意]] — 模型接入中间层为什么会长成一门产业

#### 9. Anthropic / 科学研究样本

- [[wiki/ai/claude-personal-guidance|Claude 个人指导研究]] — 个人指导型对话的行为分布与谄媚风险
- [[wiki/ai/anthropic-institute-agenda|Anthropic 研究所议程]] — TAI 四大研究支柱的高层地图
- [[wiki/ai/biomysterybench-claude-bioinformatics|BioMysteryBench]] — Claude 在生物信息学任务上的能力边界
- [[wiki/ai/2028-global-ai-leadership-scenarios|2028：全球 AI 领导权的两种情景]] — Anthropic 从 compute、蒸馏攻击与 adoption 出发，推演 2028 年民主国家能否锁定 AI 领先优势

### 知识管理（1 篇）

- [[wiki/知识管理-最佳实践|知识管理最佳实践]] — 如何让个人知识库真正发挥作用，避免只存入不取出

### 个人笔记 / 面试资料（7 篇）

- [[1.agent-llm-application-interview-handbook|Agent 应用开发 / 大模型应用开发面试八股手册]] — 把全库主线压缩成面试短答与 Python 工程落地表达框架
- [[2.agent-runtime-architecture-themes|Agent 运行时与架构主题综述]] — 从行为、runtime、control plane 三层压缩 Agent 架构主线
- [[3.context-knowledge-harness-themes|Context、知识基础设施与 Harness 主题综述]] — 总结 Prompt/Context/Harness、Spec、RAG、Memory、AGENTS.md 的分层关系
- [[5.eval-reliability-security-themes|Eval、可靠性与安全治理主题综述]] — 压缩评测、事故、权限、sandbox 与生产治理主线
- [[4.enterprise-platform-shifts-themes|企业落地、平台中间层与方法论转向主题综述]] — 总结企业 AI 架构、平台层膨胀与组织转型
- [[6.agent-application-interview-master-review|Agent 应用开发面试长篇总复习册]] — 在四篇主题综述之上重排出的长篇复习册
- [[7.genericagent-principles|GenericAgent 原理笔记]] — 围绕 context density、最小原子工具、分层记忆与运行时压缩整理出的结构化阅读笔记

## 跨主题联系

- **AI ↔ 知识管理**：对 agent 来说，不在仓库中的知识等于不存在，因此知识管理已经从“个人习惯”变成“工程接口设计”
- **Harness ↔ Spec ↔ Workflow**：当前库中最强的一条主线，不是某个模型，而是如何通过 `rules / spec / validate / review / handoff / memory` 收敛模型不确定性
- **Anthropic 官方 engineering 系列**：这一批新文章已经形成一条很强的一手主线，覆盖 context、skills、harness、tooling、eval、runtime、安全与事故复盘，适合作为后续理解 Claude/Anthropic 工程方法的基准语料
- **Claude Code ↔ OpenClaw**：两条高频对照线已经成型。前者更像 coding agent runtime 与项目上下文系统，后者更像长期 personal agent environment 与 file-first memory OS
- **Memory ↔ Runtime**：长期记忆、上下文管理、KV Cache、Sandbox、Session / Routing 都在说明 agent 系统正在长出自己的运行时基础设施
- **组织 / 产品 ↔ 工程方法**：个人工作方式迁移、团队 SOP、组织治理塌缩与平台底座建设，实际上是同一波 agent-first 转型的不同层次
- **企业总图 ↔ 单点样本**：企业 AI 应用指南给总图；OpenClaw、OpenSandbox、Agentic OS、评测 Agent、多智能体选型分别填补 runtime、控制面、安全、评测和编排层
- **生成提速 ↔ 评审与验证前移**：代码生成越快，越需要把审查和验证前移到 spec 合规、预验证、lint、test、verify 和评测 agent
- **能力升级 ↔ 平台中间层膨胀**：随着模型和 agent 能力上升，API 中转、AI Gateway、Agentic OS、Memory OS 这类中间层和底座层会继续变厚

## 待深入方向

- `Rules / AGENTS / constitution / spec / validate / verify` 这几层约束最适合各承载什么信息
- `Memory OS` 与 `工程知识底座` 的边界如何划分：用户级长期记忆、项目级知识层、运行时缓存层各自归谁
- `Claude Code` 这类项目型上下文系统与 `OpenClaw` 这类个人环境型上下文系统，哪些设计可以互相迁移，哪些不能
- `AI Gateway`、`API 中转站`、`Agentic OS`、`OpenClaw` 这几层平台化中间层的边界如何系统对照
- 多智能体模式的真实复杂度阈值：什么时候 `Single Agent First` 不再成立
- `current / historical / all` 这类时序记忆检索策略，在真实产品里怎样权衡准确性、解释性与延迟
- 评测 Agent、代码评审 Agent、Spec Reviewer 这些“判断型 agent”是否能抽象出统一治理框架
