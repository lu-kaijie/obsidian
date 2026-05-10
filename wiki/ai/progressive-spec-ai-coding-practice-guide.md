---
tags: [摘要, ai, coding, spec, workflow, review]
sources: [raw/ai/2026-05-09-progressive-spec-ai-coding-practice-guide-webclip.md]
updated: 2026-05-09
---

# 2026 年 AI 编码的“渐进式 Spec”实战指南

**来源：** [逸驹, 2026-04-02](https://mp.weixin.qq.com/s/7Lgb3GfgXKI0J9L9e9sq0w)

## 核心结论

这篇文章最值得留下的，不是再讲一遍 `Spec Coding`，而是它把一个很现实的问题说透了：`Spec-first` 如果不区分需求复杂度，很容易自己变成新的偶然复杂度。

所以作者真正提出的是一套 `渐进式 Spec` 框架：

- `Rules` 始终生效
- `Spec` 按需求复杂度逐步加深
- 简单需求不强制承担完整流程成本
- 复杂需求再引入 `spec / tasks / review / archive` 全链路

这让它和现有库里那些讲 `SDD` 正确性的文章形成了很好的互补。

## 它最强的新意是“复杂度分层”，不是“文档优先”

文章明确反对一种常见误区：

- 所有需求都走同一套重流程

作者的判断是，现实里大量需求是小改动、修 bug、改字段，如果也强制走完整 `spec -> tasks -> review -> archive`，那方法论本身就会吞掉效率。

因此它提出：

- 小需求用轻流程
- 中需求补结构化 Spec
- 高复杂需求再拉满完整控制链

这个设计很重要，因为它把 `Spec` 从“教条规则”改成了“按本质复杂度加载的控制面”。

这和 [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] 很一致：重点不是流程越重越好，而是让治理重量和复杂度相匹配。

## 这篇比一般 SDD 文章更像“实际团队作战手册”

文章给出的不是抽象口号，而是一整套可执行骨架：

- `rules/` 负责项目级稳定约束
- `knowledge/` 负责按需加载领域知识
- `agents/` 负责主 prompt 与 reviewer 分工
- `changes/` 负责每个需求的 `spec/tasks/log`
- `archives/` 负责完结后的沉淀

再加上命令式工作流：

- `/propose`
- `/apply`
- `/fix`
- `/review`
- `/archive`
- `/knowledge`

这使它比很多 “写个 spec 就好了” 的文章更具体，因为它已经开始定义 AI 编码团队的操作协议。

## “编排层 + 执行层” 是另一个很有保留价值的点

文章还提出了一种两层 AI 架构：

- `编排层 AI`：强模型，负责研究、提案、Spec 生成、审查
- `执行层 AI`：编码模型，负责读写代码、跑命令、修编译

这个拆法很实用，因为它承认：

- 做决策和做实现不是同一种能力
- 全程用最强模型太贵
- 全程用便宜编码模型做规划又不稳

所以分层后，既能控制成本，也能把角色边界拉清楚。

这和 [[wiki/ai/claude-code-qoder-cli-advanced-practice|Claude Code 与 Qoder CLI 的工作流抽象]]、[[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] 可以连起来看：前者讲 CLI / subagent / skills 的控制面，这篇讲团队内部如何把不同模型放进不同职责位。

## Review 设计比很多实践文章更严谨

文章的 review 不是一句“让另一个 agent 再看看”，而是拆成了两个独立 reviewer：

- `Spec Compliance Reviewer`
- `Code Quality Reviewer`

并且强调：

- reviewer 与实现者上下文隔离
- 不信报告，只信代码
- Spec 合规通过后，才进入代码质量审查

这个设计的价值在于，它把“审查”重新定义成针对不同失败模式的独立控制层，而不是一次泛化的 AI review。

这和 [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] 是很好的互补：那篇偏 benchmark 和规则治理，这篇偏需求驱动开发链路里的 review 分工。

## 我的判断与保留

我认为这篇最该保留的是两个具体判断：

- `Spec` 应该按复杂度渐进加载，而不是一刀切
- AI 编码系统应该区分 `编排` 与 `执行` 两层职责

相比很多 `SDD` 文章，它更像一份实际团队的控制协议，而不是方法论宣传。

它的局限也有：

- 部分模型排行和能力判断有时效性
- 整套框架仍然偏 Java / 业务后端团队语境
- 很多收益来自实践经验压缩，不是严格实验对照

但这不影响它作为 `Spec-heavy 但反官僚` 的实践样本被保留。

## 阅读评级

🔴 建议深读 — 如果你已经接受 `Spec` 有用，但又担心它把 AI 编码流程做重了，这篇非常值得看。它真正补上的，是“如何让 Spec 成为控制层，而不是新的流程负担”

## 与其他页面的关联

- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 那篇讲 Spec 为什么重要，这篇讲 Spec 应该如何按复杂度渐进加载
- [[wiki/ai/ai-coding-practice-vibe-coding-to-sdd|AI 编码实践：从 Vibe Coding 到 SDD]] — 那篇是团队从补全到规则再到 SDD 的演进复盘，这篇更像一套成熟后的操作手册
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — OpenAI 那篇强调仓库约束和记录系统，这篇补项目内 `rules + changes + reviewer` 的流程控制层
- [[wiki/ai/reliable-ai-assistant-skill-workflow-spec-coding-practice|高可靠 AI 助手的 Skill 与 Workflow]] — 一篇偏 workflow 编排与可靠性，这篇偏需求开发流程和 reviewer 分层
- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — 一篇偏代码评审治理，这篇偏把审查前移到 Spec 合规与实现偏差检测
