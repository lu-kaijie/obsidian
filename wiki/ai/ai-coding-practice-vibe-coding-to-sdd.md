---
tags: [摘要, ai, coding, sdd, workflow, rules]
sources: [raw/ai/2026-05-08-ai-coding-practice-vibe-coding-to-sdd-webclip.md]
updated: 2026-05-08
---

# AI 编码实践：从 Vibe Coding 到 SDD

**来源：** [式遂, 2026-01-23](https://mp.weixin.qq.com/s/W6-e-uSPcGCqQAXxx_PDgA)

## 核心结论

这篇文章的价值在于，它不是直接鼓吹 `SDD`，而是完整复盘了一条更真实的团队演进路径：

- 先从代码补全和单方法改写获益
- 再进入 `Agentic Coding`
- 然后因为风格漂移、团队协作和上下文不足，引入 `Rules`
- 最后探索 `SDD / spec kit`
- 实际落地时采用混合策略，而不是教条式全量切换

所以它更像一篇“AI 编码工程化成熟路径”案例，而不是单一方法宣传文。

## 从局部提效到整体生成

文章把早期收益说得很具体：

- 对象构建、模型转换这类重复代码，补全能显著减少键盘输入
- 单方法重构能提升局部代码质量

但这阶段的局限也很清楚：

- 只能优化局部
- 不理解整体业务和架构
- 对跨模块需求基本无能为力

这也是很多团队从 `copilot` 走向 `agent coding` 的共性起点。

## Agentic Coding 的红利与问题

进入 agent coding 后，开发效率继续提升，因为 AI 能一次性生成数据服务、VO、构建器等完整功能片段。

但团队很快遇到几个典型问题：

- 同类功能反复生成时，代码延续性差
- 风格和存量代码不一致
- 不同开发者 prompt 质量差异大，团队协同性差

作者的诊断很到位：问题不是“模型不够强”，而是 AI 缺少项目规范、领域知识和历史经验，每次都像在零基础生成代码。

## Rules 是第一层工程化

文章最实用的一部分，是把 `Rules` 当作 AI 的“项目规范注入层”。

团队构建了几类规则文件：

- `code-style`：命名、判空、日志、注解规范
- `project-structure`：包结构、类命名、模块边界
- `features`：数据服务、模块构建器等具体实现模式

同时为每个需求补一份轻量技术方案，定义：

- 业务定义
- 模块对象
- 数据服务
- 模块构建器

这一层带来的收益非常明确：

- 生成代码的一致性提升
- 技术方案编写成本下降
- code review 效率提升
- 新人上手更快

这和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 里“短 AGENTS + 仓库内真实约束”是同一方向，只是这里更偏业务项目内部规范。

## 为什么团队继续往 SDD 走

即便有了 Rules，文章仍然指出几类残留问题：

- AI 对业务语义理解仍然有限
- 测试质量不稳定
- 文档容易滞后
- 复杂依赖关系处理不优雅

所以团队开始尝试 `SDD`，核心理念是：

- 规格是唯一真理源
- 设计先于实现
- 可测试性内建
- 文档不再和代码分离漂移

这里比已有页面更具体的一点，是它给出了 `spec kit` 的工作流分层：

- `constitution`
- `specify`
- `plan`
- `tasks`

以及 `.specify/`、`specs/`、`req/` 的文件体系。

## 真正重要的不是 SDD 理想，而是混合落地判断

这篇最成熟的地方恰恰在于没有把 SDD 神化。文章最后的实际结论是：

- `SDD` 理念先进
- 但落地门槛高
- 工具链不够成熟
- 历史代码集成困难

所以团队当前采用的是融合策略：

- 轻量技术方案模板作为输入
- `Rules` 做严格约束
- `Agent Coding` 负责高效实现
- AI 自动汇总架构文档

这比“全仓 spec-first”更现实，也更适合大多数已有存量代码团队。

## 这篇和现有页面的差异

和 [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] 相比，这篇更偏：

- 单团队在存量 Java 业务里的逐步演进
- `Rules` 文件体系建设
- `spec kit` 的实际尝试细节
- 为什么最后停在混合策略，而不是彻底 spec-first

所以两篇是互补，不是替代。

## 阅读评级

🟡 值得追问 — 如果你关心的是“团队怎样从 vibe coding 走向更可维护的 AI 编码流程”，这篇很有参考价值。它最有价值的不是提出新概念，而是展示了一条现实可行的中间道路

## 与其他页面的关联

- [[wiki/ai/ai-fullstack-dev-practice-harness-sdd-multi-repo|Harness + SDD + 多仓管理：AI 全栈开发实践]] — 那篇偏全栈和多仓协作，这篇偏单团队在存量业务中的规则化与 SDD 尝试
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — OpenAI 那篇强调仓库级约束和记录系统，这篇给出业务团队如何把这些约束落到 rule files 和技术方案模板
- [[wiki/ai/ai-engineering-vs-traditional-engineering|AI 工程 vs 传统工程：道法术中的变与不变]] — 这篇可以看作“法”和“术”层面的具体样本：如何用确定性规范去收敛 AI 编码的不确定性
- [[wiki/ai/ai-code-review-aacr-bench|AI 代码评审实践与 AACR-Bench]] — 前者偏生成侧约束，这篇偏生成后的质量治理，两者合起来才是完整工程闭环
