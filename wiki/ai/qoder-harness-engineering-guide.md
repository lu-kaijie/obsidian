---
tags: [摘要, ai, harness, coding, validation, workflow]
sources: [raw/ai/2026-05-09-qoder-harness-engineering-guide-webclip.md]
updated: 2026-05-09
---

# Qoder 工程实践：Harness Engineering 指南

**来源：** [泮圣伟, 2026-04-03](https://mp.weixin.qq.com/s/Et3WwNtEXEgxjaQHrQFDyQ)

## 核心结论

这篇文章最值得保留的，不是再强调一次 `Harness Engineering` 很重要，而是它把 Harness 讲成了一套非常具体的仓库内执行系统：

- 短 `AGENTS.md` 做导航
- `docs/` 做按需加载知识层
- `scripts/` 做机械化验证层
- `harness/` 做任务状态、轨迹和经验层

换句话说，它不是在讲抽象方法论，而是在回答：如果你真要把仓库变成 Agent 的“操作系统”，最小可用结构应该长什么样。

## 这篇最强的点是“让 Agent 先问能不能做”

作者提出的关键转向不是“写完再修”，而是：

- 对高风险动作先预验证

例如：

- 新建文件放在哪层是否合法
- 新增 import 是否违反依赖方向

这和传统的 `写代码 -> 跑 lint -> 修错` 不同。文章认为很多 agent 翻车，不是因为它不会修，而是因为它太晚才知道自己一开始就走错了方向。

所以 Harness 的价值在于把一部分错误，从 `事后发现` 前移到 `动手前拦截`。

这点很关键，因为它把 “AI coding 的可靠性” 从 prompt 优化问题，改写成了 `预防性验证设计` 问题。

## 它把仓库拆成四层，结构很清晰

文章给出的目录骨架很有参考价值：

- `AGENTS.md`：约 100 行的导航地图
- `docs/`：架构、开发命令、产品语境、设计文档、执行计划
- `scripts/`：依赖检查、质量规则、端到端验证、统一 validate 管道
- `harness/`：任务状态、trace、经验教训

这个结构之所以有价值，是因为它明确区分了四类职责：

- 知识入口
- 真实知识
- 机械执法
- 运行痕迹与学习

和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 相比，OpenAI 那篇更偏原则，这篇更像“你在项目里到底要放哪些文件和脚本”。

## “机械执法层” 比 prompt 更关键

文章最值得记的一句潜台词是：

- 规范如果只是文档，它仍然依赖模型理解
- 规范如果变成 linter、validate、verify，它才真正从建议变成约束

所以它强调的不是给 agent 更多说明，而是给 agent 更多不会遗忘的边界：

- `lint-deps`
- `lint-quality`
- `validate.py`
- `verify` skill

这和 [[wiki/ai/business-logic-collapse-ai-agent-era-what-to-build|业务逻辑的“坍塌”]] 很能对上：当应用层一部分复杂性转移到治理层时，真正重要的是这些验证和控制机制。

## Verify 层的强调很有价值

文章把验证链拆成：

- `build`
- `lint-arch`
- `test`
- `verify`

其中 `verify` 被单独提出来，是因为作者认为：

- 编译通过不等于功能正确
- 测试通过也不一定覆盖真实用户路径

所以应该把核心用户路径编码成可执行验证脚本，形成项目级端到端验证闭环。

这点比很多 AI coding 文章更成熟，因为它不把“测试过了”误当成最终正确性。

## 我的判断与保留

我认为这篇最值得留下的是三个非常具体的实践判断：

- `AGENTS.md` 应该短，做索引，不做百科全书
- 架构与质量规则应该脚本化，而不是停留在文档层
- 对高风险动作做预验证，往往比事后修错更省上下文和 tool call

它和已有 Harness 页面不是重复关系：

- [[wiki/ai/harness-engineering-codex-agent-first|Codex 那篇]] 更像一手原则样本
- [[wiki/ai/harness-engineering-ai-democratization|AI 平权那篇]] 更像时代解释
- 这篇则更像具体仓库搭建指南

它的局限是：

- 明显偏工程实践宣言，很多收益没有严格量化
- 更适合有一定仓库治理能力的团队，不是拿来即用的通用模板

但作为 “Harness 到底如何落在 repo 里” 的样本，它很有保留价值。

## 阅读评级

🔴 建议深读 — 如果你已经认同 Harness 方向，但还不清楚应该把哪些约束、脚本、文档和验证层真正放进仓库，这篇值得细看。它最有价值的是把理念收缩成了可执行仓库结构

## 与其他页面的关联

- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — 那篇给出 agent-first 原则，这篇把原则压缩成具体的 repo 结构和验证动作
- [[wiki/ai/harness-engineering-ai-democratization|Harness驾驭工程是AI平权的必经之路？]] — 那篇解释为什么 Harness 会成为共识，这篇解释共识落地后仓库里应该出现什么
- [[wiki/ai/progressive-spec-ai-coding-practice-guide|渐进式 Spec 实战指南]] — 一篇偏需求流程与 reviewer 分层，这篇偏预验证、validate 管道和仓库治理
- [[wiki/ai/engineering-knowledge-engine-harness-foundation|工程知识引擎]] — 那篇偏知识底座，这篇偏执行和验证底座
