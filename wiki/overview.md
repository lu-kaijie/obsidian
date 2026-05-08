---
tags: [概览]
updated: 2026-05-08
---

# Wiki 概览

知识库整体状态，由 LLM 在每次 Lint 时维护。

## 当前状态

- **主题数**：2（AI、知识管理）
- **Wiki 页面数**：11
- **已摄入资料数**：10
- **最后更新**：2026-05-08

## 主题地图

### AI（9 篇）

- [[wiki/ai/claude-personal-guidance|Claude 个人指导研究]] — 6% 对话涉及个人指导，谄媚率领域差异显著
- [[wiki/ai/anthropic-institute-agenda|Anthropic 研究所议程]] — TAI 四大研究支柱：经济扩散、威胁与韧性、AI 现实影响、AI 驱动研发
- [[wiki/ai/biomysterybench-claude-bioinformatics|BioMysteryBench]] — Claude 在生物信息学任务上接近/超越人类专家
- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理]] — 用约束、SDD 与多 Agent 并行提升 AI 全栈开发的可控性
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — OpenAI 的 agent-first 工程经验：仓库即记录系统，短 AGENTS 胜过长手册
- [[wiki/ai/ai-native-rd-organization-future|AI Native 时代：研发组织何去何从]] — 从组织设计视角讨论 execution graph、Harness/Hive Mind 双层结构与新治理
- [[wiki/ai/enterprise-ai-application-building-guide|企业 AI 应用构建指南]] — 企业 AI 应用的总架构地图：Agent、交付、MaaS、MCP、Sandbox、观测与安全
- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — AI 编码规模化之后，如何用 agent 评审、规则收敛和 benchmark 守住质量
- [[wiki/ai/opensandbox-agent-sandbox|OpenSandbox]] — 面向 AI Agent 的通用沙箱平台，强调隔离、统一协议、Kubernetes 调度与远程 runtime

### 知识管理（1 篇）

- [[wiki/知识管理-最佳实践|知识管理最佳实践]] — 如何让个人知识库真正发挥作用，避免只存入不取出

## 跨主题联系

- **AI ↔ 知识管理**：TAI 的"群体认识论"和"批判性思维退化"研究与知识管理直接相关——大规模使用 AI 是否会削弱独立思考能力
- **AI 能力迭代 → 知识库维护频率**：模型能力快速提升（如 BioMysteryBench 的突破）意味着旧结论可能需要更频繁的 lint 更新
- **工程约束 ↔ Agent 产出质量**：Harness + SDD 的实践说明，高质量 AI 产出更多依赖约束与流程设计，而不只是模型能力
- **仓库即知识接口**：OpenAI 的 Codex 实践说明，对智能体来说，不在仓库中的知识等于不存在
- **组织设计 ↔ 信息形态**：AI Native 组织的真正瓶颈不是模型弱，而是信息、系统和权限仍按“人形偏置”组织
- **生成提速 ↔ 评审重估**：代码生成越快，评审与质量治理越成为新瓶颈，AACR-Bench 和 scoped rules 正是在补这一层
- **企业架构图 ↔ 单点实践**：企业 AI 应用指南提供总图，Codex agent-first 与 AI review 实践分别填充了执行层和治理层
- **执行能力 ↔ 安全边界**：OpenSandbox 把“agent 能执行”与“agent 该被怎样隔离和约束”连接起来，补齐了 runtime 层

## 待深入方向

- TAI 的"群体认识论"研究进展 — 如何测量大规模 AI 使用对人类信念和思维方式的影响
- Claude 在其他科学领域的 benchmark 表现 — 生物信息学之外，AI 在哪些研究领域能超越人类
- 个人指导类对话的长期影响 — 用户实际决策结果追踪（当前研究缺失）
- Harness 思维在不同代码库中的适用边界 — 什么情况下“模仿已有实现”会抑制更优设计
- agent-first 工程的泛化边界 — OpenAI 这种重投入控制系统，在哪些团队规模和项目复杂度下仍成立
- AI Native 组织的治理边界 — 如何同时维持执行透明度和创新所需的“生产性 ego”
- OpenSandbox 与其他 sandbox / remote runtime 的对比 — 当前已补到原文，但还没横向比较不同实现路线
