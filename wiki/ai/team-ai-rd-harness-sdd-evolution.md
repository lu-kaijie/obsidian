---
tags: [摘要, ai, coding, harness, sdd, organization]
sources: [raw/ai/2026-05-10-team-ai-rd-harness-sdd-evolution-webclip.md]
updated: 2026-05-10
---

# 告别“氛围编程”：基于 Harness 治理和 SDD 的团队级 AI 研发范式演进与实践

**来源：** [王树新, 2026-05-07](https://mp.weixin.qq.com/s/-_IBJFuXpvoqMJxL9oaEJQ)

## 核心结论

这篇文章最值得保留的地方，是它直接指出了一个常见幻觉：`出码率上升，不等于交付提效`。如果不把 SDD 和 Harness 拉进全流程，团队只会得到更多 AI 代码，而不是更短交付周期。

## 它抓住了“团队级”这一层

文章把问题放在需求、评审、方案、开发、测试、联调、上线的整条链上，而不是停在编码环节。这和 [[wiki/ai/agent-productivity-paradox-collaboration-bottleneck|Agent 时代的生产力悖论]] 可以形成很强的互补：两篇都在说明瓶颈从单人写码迁移到了协作与链路治理。

## SDD + Harness 在这里被放成因果关系

文中的核心组合是：

- SDD 负责把模糊意图编译成结构化规范
- Harness 负责让 AI 在受控环境里执行、验证和修正

这个定位和 [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]]、[[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] 之间关系很清晰，适合当团队级 adoption 页。

## 知识库三层和单一事实来源值得保留

文章把知识按项目层、技术层、资产层组织，并强调顶层 README 作为单一事实来源。这和当前知识库已经积累的“仓库即记录系统”主线完全一致。

## 我的判断与保留

我认为这篇最值得留下的是三个点：

- 团队级 AI 研发的关键指标不该只看出码率
- SDD 和 Harness 要一起进入需求到部署的全链路
- 真正的落地点是知识库、规则、验证和交付系统，而不是更会写 prompt

## 阅读评级

🔴 建议深读 — 如果你关心的不是个人 AI coding 技巧，而是团队为什么“AI 写得更多了却没快多少”，这篇很值得看

## 与其他页面的关联

- [[wiki/ai/agent-productivity-paradox-collaboration-bottleneck|Agent 时代的生产力悖论]] — 一篇偏组织结构，一篇偏团队研发链路
- [[wiki/ai/spec-driven-development-redefining-ai-coding|Spec-Driven Development 重新定义 AI 编程]] — 那篇讲 SDD 总论，这篇讲团队 adoption
- [[wiki/ai/ai-agent-harness-engineering-real-projects|从玩具到生产力：AI Agent 的 Harness Engineering]] — 都强调从局部写码提效走向系统级控制面
