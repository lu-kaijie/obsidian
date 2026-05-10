---
tags: [摘要, ai, coding, workflow, sdd, team]
sources: [raw/ai/2026-05-09-ai-native-rd-paradigm-doc-driven-evolution-webclip.md]
updated: 2026-05-09
---

# AI 原生研发范式：文档驱动演进

**来源：** [无岳, 2026-02-04](https://mp.weixin.qq.com/s/WoEetgbDkNidf7Flmg8dUQ)

## 核心结论

这篇文章最值得保留的地方，是它没有把 `SDD` 讲成抽象理念，而是把 AI 编码时代最典型的三类工程失序明确点出来：

- `上下文腐烂`
- `审查瘫痪`
- `维护断层`

然后再把 `Spec` 定位成这些问题的“存档点”或“锚点”。这个说法很实用，因为它直接回答了一个一线团队最常见的问题：为什么明明 AI 写得更快，工程却反而更容易失控。

## 这篇比前一篇更偏团队 SOP

和 [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] 相比，这篇不是强调个人角色迁移，而是更强调组织如何形成一套可反复执行的协作协议。

它的核心判断是：

- 不是缺更强的模型
- 而是缺一套 AI 时代的研发方法
- 重点不在单个工具有多强，而在团队有没有统一输入标准、产出约束和 review 门禁

这个判断和 [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] 很契合，只是这里给的是更轻量、更手工的落地版本。

## 它对 SDD 的一个好比喻

文中把 `SDD` 解释成 `Vibe Coding` 的“存档点”很有启发。

这个比喻背后的意思是：

- 代码可以反复重生成
- 但只有当 `Spec` 稳定时，重生成才不是抽奖
- 出问题时，应该回到文档锚点，而不是在漂移的实现上不断打补丁

这让 `SDD` 从“文档驱动开发”变成了一种上下文稳定机制，而不只是流程规范。

## 最有价值的是它把轻量版和团队版分开了

很多 SDD 文章的问题在于默认一上来就全量流程化。这篇更务实，它区分了：

- `Lite / Vibe Mode`：个人开发、原型验证，只需要一份简要 `Task Spec`
- `Team Mode`：多人协作、复杂业务，需要完整 `Requirements -> Interface -> Implementation` 链路

这个分层很重要，因为它避免把 `SDD` 直接做成高摩擦官僚流程，也更符合真实团队逐步升级的路径。

## RIPER 工作流是这篇的落点

文章后半段给出的 `Initialization + RIPER` 流程，是它相对独特的贡献：

- `Research`：先反向复述需求，锁定意图
- `Innovate`：先做方案推演和互问互答，不急着写代码
- `Plan`：把实施路径、文件、接口、mock 数据写清楚
- `Execute`：按步骤让 AI 分段实现，并做硬性门禁
- `Review`：开新对话或换模型，基于 Spec 和 Diff 做“法医式审查”

这套流程本质上是在把 AI 编码从“连续聊天”改成“分阶段门禁式推进”。它和 [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] 的外部循环控制很互补：那篇偏“如何持续推进”，这篇偏“每一轮推进之前和之后怎样设门”。

## Manual SDD 的立场值得记一下

文中对 OpenSpec、Spec-Kit 这类工具的态度也比较成熟：不是否定它们，而是认为很多团队更需要先把 `SDD` 养成习惯，再谈把它工具化。

这个判断可以压缩成一句话：

- 工具框架是在把 `SDD` 变成流程
- 这篇更想把 `SDD` 变成日常习惯

这个差异很关键，因为很多团队失败不是败在方法错，而是败在一开始引入的流程和工具摩擦过大。

## 我的判断与保留

这篇的价值在于把 `SDD` 从“文档更完整”提升为“工程稳定性控制手段”，尤其对 `review paralysis` 的描述非常贴近现在 AI 写大 diff 的真实困境。

但它也有边界：

- 依旧偏倡导和经验总结，缺少严格对照数据
- `RIPER` 更像工作法，而不是被广泛验证的行业标准
- 对成熟大团队来说，最终还是要和 lint、CI、测试门禁、权限策略结合，不能只靠 SOP 文档

所以更稳妥的理解是：这是一份很好的“轻量 AI-native 工程手册”，不是完整控制系统本身。

## 阅读评级

🟡 值得追问 — 如果你关心的是如何让 AI 编码从个人提效变成团队可维护的协作流程，这篇很值得保留。它最有价值的不是再讲一遍 `SDD`，而是把 `context decay / review paralysis / maintenance gap` 三类失序问题和轻量门禁式 SOP 连接起来

## 与其他页面的关联

- [[wiki/ai/transition-from-traditional-to-llm-programming|从传统编程转向大模型编程]] — 前者偏个人工作习惯迁移，这篇偏团队 SOP 与门禁设计
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 那篇偏 SDD 方法总论，这篇偏轻量落地流程与 Save Point 比喻
- [[wiki/ai/harness-engineering-codex-agent-first|在智能体优先的世界中利用 Codex]] — OpenAI 那篇偏完整控制系统，这篇偏低摩擦 Manual SDD
- [[wiki/ai/react-to-ralph-loop-agent-iteration|从 ReAct 到 Ralph Loop]] — 一篇偏持续迭代控制，一篇偏每轮迭代前后的文档门禁与审查流程
